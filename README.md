# https://evgeniy.tech/ (ex www.trueadmin.ru)

This is a Jekyll-based website for https://evgeniy.tech/.

## Running locally

To run the website locally, you need to have Ruby and Bundler installed.

1.  **Install dependencies:**

    ```bash
    bundle install
    ```

2.  **Run the local server:**

    ```bash
    bundle exec jekyll serve --config _config.yml,_config_dev.yml
    ```

3. **In Docker**

```bash
docker run --rm -it -p 4000:4000 -v "$PWD":/srv -w /srv ruby:3.3 bash -c "bundle install && bundle exec jekyll serve --host 0.0.0.0 --config _config.yml,_config_dev.yml"
```

4. **Open**
    The website will be available at [http://127.0.0.1:4000](http://127.0.0.1:4000).

