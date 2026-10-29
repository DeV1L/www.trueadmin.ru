---
title: "Reading kindergarten emails with DeepSeek and Telegram"
date: 2026-10-28
---

<!--
EDITOR NOTES (delete before publishing)

Placeholders to fill:
  [SCREENSHOT: ...]   a screenshot goes here
  [NUMBER: ...]       a real number from the logs or the Gmail labels
Everything else is written from the repository as of 2026-10-28.

Medium import lessons from the k3s post:
  - No code inside a link. Write [prompt.md](url), not [`prompt.md`](url).
  - The two diagrams are PNG files in /images, drawn in draw.io. The
    .drawio sources are next to them.
  - Code blocks get dropped by the importer. Re-add them in the editor.
  - Medium has two heading sizes. H2 = chapter, H3 = subsection (Ctrl+Alt+2).
-->

## Introduction

My two-year-old daughter attends a kindergarten in Argentina. There is a rule that all communication between staff and parents happens via email only. They send everything there: important and unimportant news, schedule updates, holiday greetings, payment reminders and so on. So it's a mix of noise and things that I wouldn't want to miss.

I don't speak Spanish, so I have to translate every email. And here is the problem. They tend to send emails with little text (or even without it) but add images with text on them. A built-in Gmail translator does not help with this. Here is an example.

![A typical email](/images/kindergarten-email-checker-email-example.png)


So, I decided to build a tool that processes these emails for me, classifies them and sends a short summary in Russian to a Telegram group where my wife and I can comfortably read them.

## Initial conditions and constraints

Before writing code, I wrote down what the tool must and must not do:

- **No access to a personal mailbox:** A tool that reads my whole inbox with an LLM is not something I want to run.
- **Simple architecture:** One container, no overhead. It should runs on my Always Free k3s from the [previous post]().
- **One email -> One Telegram message:** Short summary in Russian, with an urgency and a category labels.
- **Process Images and PDFs:** Most of the content lives there, so the model has to see them.
- **Minimize costs:** Don't send too much to an LLM, use a cheap one.

## Architecture

![Architecture](/images/kindergarten-email-checker-architecture.png)
<!-- Mermaid source in docs/ARCHITECTURE.md, "System overview". Export as PNG. -->

The parts:

- **A forwarding filter** in my personal Gmail. It matches the kindergarten addresses and forwards each email to a dedicated Gmail account. This is the whole "integration" with my mailbox: a filter that I set up once in the Gmail settings.
- **A dedicated Gmail account** that only the tool uses. It logs in over IMAP with an app password. The Gmail labels are the only state of the tool: `INBOX` is the queue, `Processed`, `Ignored` and `Failed` record the outcome.
- **One Python process in a Docker container.** Every 10 minutes it connects to the mailbox, takes the unseen emails, processes them one by one and logs out.
- **DeepSeek** with the model `deepseek-flash`. It accepts images, it is cheap, and it speaks the Anthropic Messages API format, so I use the `anthropic` SDK with a custom base URL.
- **A Telegram bot** that posts to a private group with two members: my wife and me.

The container makes outbound connections only: IMAP to Gmail, HTTPS to DeepSeek and Telegram. No ports, no webhooks, no public endpoint.

### Processing one email

1. Fetch the unseen emails in `INBOX` without marking them as seen.
2. Compare the sender with the allowlist. A stranger goes to `Ignored` without a post.
3. Extract the content: the text body, the text of PDFs with a text layer, page images of scanned PDFs, the photos. Images under 20 KB are skipped, they are logos and signatures.
4. Call the model. It returns the summary as a tool call, validated against a Pydantic schema. If the answer is unusable, one retry with the validation error. If the second answer is unusable too, the post carries the original text instead of a summary.
5. Post to Telegram.
6. Move the email to `Processed`. This is the commit point.

Telegram is called before the move. A crash between the two gives one duplicate post, never a lost email. I prefer that direction.

### What a post looks like

```text
🟡 Medium · 🎉 Мероприятие · ⏰ Срок: пт, 9 окт · 📅 Событие: вт, 20 окт 09:30
Экскурсия в Ботанический сад

<three short sentences in Russian>

Что сделать: <one line, what the parent must do>
📎 Вложения: menu.pdf, autorizacion.pdf

Тема: <subject> · От: <sender> · Отправлено: пт, 2 окт 2026 14:52
```

The first line has everything I need to decide whether to read on: urgency, category, deadline, event date. The attachments are listed by name but never uploaded. The original email stays in the dedicated mailbox, one click away.

[SCREENSHOT: the Telegram group with two or three posts, one of them from the email above]

## Interesting details

### Gmail labels as the database

I did not want a database for a few emails a day. The tool has nothing on disk. An email is in exactly one label, and the label says what happened to it. Moving the email is the only write the tool ever does.

![Gmail labels as the state](/images/kindergarten-email-checker-labels.png)

The bonus: the Gmail web UI is the admin interface. I open the `Failed` label and see what went wrong. I drag an email back to `INBOX` and the tool processes it again at the next poll. No commands, no SQL.

### The model call

DeepSeek serves the Anthropic Messages API format at `https://api.deepseek.com/anthropic`. The whole call is one request:

```python
response = client.messages.create(
    model=settings.llm_model,
    max_tokens=settings.llm_max_tokens,
    system=SYSTEM_PROMPT.replace("{TOOL_NAME}", tool_name),
    messages=messages,
    tools=[tool_definition(settings)],
    # DeepSeek rejects a forced tool_choice with thinking on. prompt.md asks for the call.
    tool_choice={"type": "auto"},
    thinking={"type": "enabled", "budget_tokens": 8000},  # DeepSeek ignores the budget
    output_config={"effort": settings.llm_effort},
)
```

Two things in it took a while to find out.

**Thinking and a forced tool call do not go together.** I wanted thinking on: the emails have dates like "el viernes" that must be resolved from the sent date, and the urgency needs a small judgement. I also wanted a structured answer, so the summary is the input of one tool, `report_summary`, with a JSON schema generated from a Pydantic model. The obvious way to make the model call that tool is `tool_choice={"type": "tool", "name": "report_summary"}`. DeepSeek answers: "Thinking mode does not support this tool_choice". So `tool_choice` is `auto`, and the system prompt tells the model to call the tool exactly once. [CONFIRM: in three weeks it never answered in prose instead of calling the tool.]

**Invalid JSON looks like an empty call.** When the model writes tool arguments that are not valid JSON, DeepSeek does not report an error. It returns the tool call with the input `{}`. Pydantic rejects that, and the tool sends the error back as a `tool_result` block with the full previous response, thinking blocks included. [CONFIRM: The second answer has been fine every time.]

### "null" is not null

The schema has four optional fields: the action, the deadline, the event date and the event time. The model sometimes writes the string `"null"` instead of a JSON null. The fix is three lines: a validator that treats `""`, `"null"` and `"none"` as no value. It is cheaper than a retry.

### Urgency is about time, not about the topic

My first prompt said: `high` for anything urgent, cancelled, closed, or about health or safety. The model did what I asked. Every health notice became red, including "if your child has symptoms, keep them at home". That is a rule, not an alarm.

Now the prompt asks one question: how soon must the parent do something?

- `high`: act within two days, or attendance changes within two days (a closure, an early pickup).
- `medium`: act later, or an event with the child later.
- `low`: nothing is required. General rules and conditional instructions are `low`.

The topic goes into the category instead: payment, event, closure, health, and so on. The category is shown, nothing depends on it.

### Images and PDFs

DeepSeek accepts images but not documents. So the tool does the paperwork:

| Input | What the model gets |
|---|---|
| HTML body | The text/plain part if there is one, otherwise html2text |
| PDF with a text layer | The text, extracted with pypdfium2 |
| PDF without a text layer | Each page rendered at 110 dpi, at most 10 pages |
| JPEG, PNG, GIF, WebP | As is, when larger than 20 KB |
| Anything else | Not sent. The file name is listed in the prompt and in the post |

A "text layer" is present when the average page has more than 200 characters. Below that, it is a scan.

### Dates and the timezone

The prompt gives the model the sent date, the weekday and today's date, all in the Buenos Aires timezone. "Mañana" in an email sent on Friday evening is Saturday, not the day I read it. The post shows the deadline and the event with the weekday, which is what a parent looks at first.

### The bot token in the logs

A small one. The Anthropic SDK and the tool itself use `httpx2`, and at the `INFO` level it logs the URL of every request. Telegram URLs contain the bot token. A log formatter replaces the token in every line, tracebacks included. The path reads `/bot<token>/`.

## Deployment

CI builds the image and pushes it to `ghcr.io/dev1l/kindergarten-email-checker`. The host pulls the image, it never builds. Tags: `latest` for `main`, the branch name for a branch build, `sha-<commit>` for every build.

There are two ways to run it:

- **Docker Compose on a VPS.** Two files on the host: `compose.yaml` and `.env`. Update: `docker compose pull && docker compose up -d`.
- **Kubernetes with a Helm chart.** This is what I use, on the free k3s cluster from the previous post. A GitHub Actions workflow runs `helm upgrade --install` on a manual start with an image tag.

At start, the tool checks the three services once: Telegram must accept the bot token and find the group, Gmail must accept the app password, DeepSeek must accept the API key and report that the account can pay. If one check fails, the process stops with exit code 1, and Kubernetes or Docker restarts it with a growing delay. Without this check, a wrong password would send every email to `Failed`.

On Kubernetes, `/readyz` turns 200 after the start check, and the deployment uses `RollingUpdate` with `maxUnavailable: 0`. A new pod with wrong credentials never becomes ready, and the old pod keeps working. The whole thing fits in a few tens of MB of RAM.

## Three weeks in production

The tool went live in mid-October 2026. Some numbers:

- [NUMBER: emails processed]
- [NUMBER: posts without a summary, the fallback]
- [NUMBER: emails in Failed]
- DeepSeek cost: [NUMBER] for the period. The estimate was "cents per month", and it holds.

And the problems I found, in the order I found them.

**IMAP IDLE was the wrong choice.** The first version kept one IMAP connection open and waited with IDLE, so a post appeared seconds after the email. Gmail and the network closed that connection a few times every day. Nothing was lost, the tool reconnected, but every drop was a warning in the log and I kept looking at it. I replaced IDLE with a plain poll: a new connection every 10 minutes, process, log out. The cost is a delay of up to 10 minutes. For a kindergarten, that is real time.

**Gmail shows the MOVE capability only after login.** The IMAP library reads the capabilities before login, sees no MOVE, and falls back to copy and delete. With some Gmail settings, "delete" means "move to Trash". With that, a "processed" email can end in Trash. The fix is to read the capability list again after login.

**The summaries were too long.** "Two to four sentences" in the prompt became four long sentences with the address of the school and a phone number in each. Now it is at most three short sentences, under 350 characters, and no addresses or phone numbers unless they are needed for the action.

**The urgency rule.** Described above. The topic does not decide the urgency any more.

**Memory.** A review found that a poll fetched all unseen emails at once, with their attachments. After a weekend, or after a restart with a backlog, that is many photos in memory at the same time on a node with 1 GB. Now the tool fetches one email at a time. One email with its attachments is in memory, then the next.

**Graceful shutdown.** `docker stop` or a pod deletion could hit the tool between the Telegram post and the move to `Processed`, and the email would get a second post after the restart. Now `SIGTERM` sets a flag, the tool finishes the current email, logs out and exits with code 0. The stop grace period is 180 seconds, enough for a model call with one retry.

**Rolling updates give a short overlap.** After the new pod passes its start check, both pods poll until the old one stops, at most 10 seconds. An email that arrives in that window gets two posts. I accepted this one. The alternative is `Recreate` with a gap, and a gap is worse than a duplicate.

What did not happen: [CONFIRM: no email was lost, and the fallback post with the original text never fired / fired N times].

[SCREENSHOT: an example where the summary got a tricky email right, for example a relative date or a scanned PDF]

## What is next

An evaluation of different models. I have about 100 emails from the kindergarten by now, with the summaries that `deepseek-flash` wrote for them. I will turn them into a dataset: the email as the model sees it, and the expected urgency, category, deadline and event date. Then I can run the same prompt against other models and compare. The questions are simple: which models get the dates and the urgency right, what does a wrong answer look like, and how much does each cost per email. The dataset also gives me a regression test for the prompt, which I change by feel today.

That will be the next post.

The code is on GitHub: [DeV1L/kindergarten-email-checker](https://github.com/DeV1L/kindergarten-email-checker).
