---
title: "A free Kubernetes cluster on Oracle Cloud with k3s and Pulumi"
date: 2026-10-19
---

<!--
DRAFT 1. Publishing notes:
- Medium subtitle: "Two 1 GB VMs from the Always Free tier, one Pulumi program, zero dollars."
- Publish on evgeniy.tech first, then import into Medium (medium.com/p/import) so the canonical
  URL points to the blog.
- Medium has no tables and no mermaid. The post uses bullet lists and images only.
- Images to make, in images/, with the .drawio source next to the PNG:
  1. k3s-nodes.png: screenshot of `kubectl get nodes -o wide`, both nodes Ready. Hero image.
  2. oracle-cloud-k3s-infra.png: Draw.io, based on the flowchart in docs/ARCHITECTURE.md.
  3. oracle-cloud-k3s-pulumi.png: Draw.io, based on the flowchart in docs/PULUMI.md.
-->

## Two tiny VMs

Oracle Cloud has an Always Free tier. The best part of it is two always free super-tiny `VM.Standard.E2.1.Micro` instances. Each has one CPU (actually, [OCPU](https://blogs.oracle.com/cloud-infrastructure/vcpu-and-ocpu-pricing-information)) and 1 GB of RAM.

But what can you do with two 1 GB machines?

Of course, turn them into a Kubernetes cluster!  😎 

I wrote a Pulumi project that spins up a two-node k3s cluster on Oracle Cloud, with all cloud resources it needs.

![Two nodes, both Ready](/images/k3s-nodes.png)

## Architecture

The repository is a Pulumi program in Python. It creates:

- A virtual cloud network (VCN) with a public subnet, an internet gateway and a security list.
- Two instances with Ubuntu 24.04 Minimal: the server node `k3s-server` and the agent node `k3s-agent-1`. k3s installs itself at the first boot.
- A network load balancer with the one reserved public IP (only one is available for free). It forwards port 6443 to the server node, and ports 80 and 443 to both nodes.
- An ephemeral public IP on each node, for SSH and outbound traffic.
- An OCI Vault with two secrets: the k3s join token and the kubeconfig.
- Traefik with Let's Encrypt certificates for Ingress.

Everything is inside the Always Free limits. The bill is zero.

The code is on GitHub: [DeV1L/oracle-cloud-k3s](https://github.com/DeV1L/oracle-cloud-k3s).

## How it fits together

![Infrastructure](/images/oracle-cloud-k3s-infra.png)

Traffic from the internet reaches the reserved public IP on the load balancer. The load balancer sends HTTP and HTTPS to both nodes, where Traefik takes over. kubectl goes to port 6443 and reaches the server node only.

The reserved IP sits on the load balancer, not on a node. When a node is replaced, the IP and the DNS name stay the same. A NAT gateway is not free, so each node gets its own ephemeral public IP for outbound traffic and SSH.

## The Pulumi program

![The Pulumi program](/images/oracle-cloud-k3s-pulumi.png)

The program has one stack, `default`. Every account-specific value is in `Pulumi.default.yaml`: the region, the compartment, the availability domain and the SSH public key. The main program reads the configuration once, creates a random join token and then four components:

```python
network = NetworkComponent("network", config)
vault = VaultComponent("vault", config, join_token=join_token.result)
load_balancer = LoadBalancerComponent("lb", config, subnet_id=network.subnet_id)
cluster = K3sClusterComponent(
    "cluster",
    config,
    subnet_id=network.subnet_id,
    join_token=join_token.result,
    public_ip=load_balancer.ip_address,
)
```

Each component is a `pulumi.ComponentResource` in its own module. It owns one area of the infrastructure: the network, the vault, the load balancer or the cluster. The child resources get `parent=self`, so the preview and the state show them grouped under their component. The main program connects the components through their outputs. The subnet ID goes to the load balancer and the cluster. The reserved IP goes to the cluster.

**Configuration.** One function reads the stack configuration and returns a typed `ProjectConfig` object. The components get it as an argument and never read the configuration on their own. A value lives in one of three places. The required keys are different for every account. The optional keys, like the DNS name of the API, change what the program does when they are set. Everything else is a `DEFAULT_*` constant: the k3s version, the CIDRs, the private IPs, the swap size. A key in the stack file overrides the default.

**The join token.** The token is a `RandomPassword` resource. The Python program only declares it. The Pulumi engine creates it later, so the value does not exist while the program runs. What the program holds is an `Output[str]`. An `Output` carries three things: the value, once it is known, the dependency on the resource that makes it, and the secret flag. The same token goes into the Vault secret and into the cloud-init of both nodes, and Pulumi keeps it out of the logs.

**Cloud-init.** The templates `server.yaml` and `agent.yaml` have placeholders in the form `@{name}`. The cluster component fills them inside `Output.all(join_token, public_ip).apply(...)`, encodes the result in Base64 and passes it to OCI as `user_data`. The order matters here. Pulumi creates the reserved IP before the nodes, so the server node can put the public IP and the DNS name into the API certificate at the first boot.

The cloud-init writes the k3s configuration and one bootstrap script, then runs the script. The script makes a 2 GB swap file, removes two iptables rules of the Ubuntu image, stops services the node does not need and installs the pinned k3s version. On the server node, the cloud-init also drops a Traefik configuration into the k3s manifests folder, and the script writes the kubeconfig. The script is safe to run twice. To test a change, I copy the new script to a live node and run it there.

**Resource options.** A few options protect the cluster from the program itself:

- `protect=True` on the instances. Pulumi refuses to delete a node by accident.
- `ignore_changes` on the image and the metadata. A new image or a new cloud-init does not replace a node. OCI rejects such changes anyway, and cloud-init runs only once.
- `delete_before_replace`. The fixed private IP and the limit of two instances leave no room for a second instance during a replacement.
- `refresh: always` in `Pulumi.yaml`. Every preview sees what was deleted by hand in the console.

## GitHub Actions

The repository is the source of truth, so the deployment runs from it. One workflow has two jobs. `pulumi preview` runs on every pull request and every push to `main`. `pulumi up` runs only on a manual start, with an input that must be set to true. A concurrency group allows one run at a time. A queued run waits and is not cancelled.

The Pulumi state is in an OCI Object Storage bucket, through its S3-compatible API. The workflow gets the Customer Secret Key as `AWS_*` secrets and the API signing key as a repository secret. One detail: Pulumi CLI 3.256 and older cannot log in to this bucket. The workflow pins 3.268.

## Inside Always Free

The tier has limits that shape the design. This is how the project uses them:

- Two micro instances, only in availability domain AD-1: the two nodes.
- One reserved public IP: the load balancer.
- One network load balancer: the entry point.
- One vault of type DEFAULT with software-protected keys: the join token and the kubeconfig. A virtual private vault and HSM keys cost money.
- 200 GB of block storage: the two boot volumes.
- A NAT gateway is not free, so there is none.

## Notes from running it

A few facts about OCI and k3s on 1 GB that are worth knowing before the first `pulumi up`:

- Capacity for the micro shape is not guaranteed. A deleted instance can take days to come back. The repository has a script that checks the capacity. Run it before any replacement.
- OCI rejects changes to `user_data` and SSH keys on a live instance. A cloud-init change applies to new nodes only.
- A deleted vault waits 7 to 30 days for its deletion and still counts against the limit of one. A rebuild of the cluster keeps the vault.
- The Ubuntu image rejects forwarded traffic with two iptables rules. Without the fix in the bootstrap script, pods cannot talk to each other.
- The server node uses about 600 MB of its 1 GB and some swap. After a replacement, the system pods can land on the server node. Move them to the agent node.
- Let's Encrypt allows 5 identical certificates in 7 days. The certificates live on a volume on the node of the Traefik pod, and a replacement of that node requests them again. Test rebuilds use the staging server.

## What runs on it

The cluster does real work. It runs kindergarten-email-checker, a small Python service. It reads the emails that a kindergarten sends in Spanish, writes a short summary in Russian with an LLM, and posts it to a private Telegram group. One container, one Helm chart, no database. It fits in the memory budget.

The next post is about that tool and how it got onto this cluster. A couple more small tools of this kind will follow. The honest limit: anything that needs more than a few hundred MB of RAM does not fit here.

## Try it

1. Create the OCI credentials: an API signing key and a Customer Secret Key.
2. Put the region, the availability domain and the SSH public key into `Pulumi.default.yaml`.
3. Run `pulumi up`, or start the GitHub Actions workflow.

The full guide, with every command, is in `docs/SETUP.md` in the repository.

- Repository: https://github.com/DeV1L/oracle-cloud-k3s
- Always Free resources: https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm
- The LinkedIn post: https://www.linkedin.com/posts/esavchenko_kubernetes-k3s-pulumi-ugcPost-7516691954253107200-J-x0
