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
  - The side-by-side pairs in "Post examples" are HTML figures. Medium
    imports them as four stacked images. To put a pair in one row, select
    the two images in the editor and pick the two-column grid layout.
  - The #### headings of the six stages come out as the small heading too,
    the same as H3. The numbers keep the order readable.
-->

## Introduction

My two-year-old daughter attends a kindergarten in Argentina. There is a rule that all communication between staff and parents happens via email only. They send everything there: important and unimportant news, schedule updates, holiday greetings, payment reminders and so on. So it's a mix of noise and things that I wouldn't want to miss.

I don't speak Spanish, so I have to translate every email. And here is the problem. They tend to send emails with little text (or even without it) but add images with text on them. A built-in Gmail translator does not help with this. Here is an example.

[![A typical email](/images/kindergarten-email-checker-email-example.png)](/images/kindergarten-email-checker-email-example.png)


So, I decided to build a tool that processes these emails for me, classifies them and sends a short summary in Russian to a Telegram group where my wife and I can comfortably read them.

## A starting point

Here is the general idea:

- **No access to a personal mailbox:** A tool that reads my whole inbox with an LLM is not something I want to run.
- **Simple architecture:** One container, no overhead. It should run on my Always Free k3s cluster from the [previous post](https://evgeniy.tech/2026/10/20/oracle-cloud-k3s.html).
- **One email -> one Telegram message:** A short summary in Russian, with urgency and category labels.
- **Process images and PDFs:** Most of the content lives there, so the model has to see them.
- **Minimize costs:** Don't send too much to the LLM, and use a cheap one.

## Implementation
A high-level view:

1. A forwarding filter in the personal mailbox forwards the emails from the kindergarten addresses to the dedicated mailbox.
2. Every 10 minutes, the app connects to the dedicated mailbox over IMAP with an [app password](https://support.google.com/mail/answer/185833) and checks for new emails.
3. It checks each unseen email against the sender allowlist and processes the matching ones one by one. Then it logs out.
4. The app extracts the text and the attachments from the email and sends them to DeepSeek through the Anthropic-compatible API.
5. The model returns a structured summary: a Russian title and summary, urgency, category, action, deadline and event date.
6. The app posts the summary to the Telegram group. When the email has attachments, the post lists their names. Then the app moves the email to the `Processed` label.

Gmail labels are the only state: `INBOX` is the queue; `Processed`, `Ignored` and `Failed` record the outcome. No database is needed.

[![Architecture](/images/kindergarten-email-checker-architecture.png)](/images/kindergarten-email-checker-architecture.png)
<!-- Mermaid source in docs/ARCHITECTURE.md, "System overview". Export as PNG. -->

### Processing one email

Every 10 minutes the app checks the mailbox. It opens a new IMAP connection, creates the state labels if they are missing, and asks for the unseen emails in `INBOX`. It fetches them one by one, without marking them as seen, so one email with its attachments is in memory at a time. Each email goes through six stages.

#### 1. Allowlist

The app reads the `From` header and compares it with the allowlist. Gmail forwarding keeps this header untouched, so the forwarded copy still shows the kindergarten address. If the sender is on the list, the email moves on to the next stage. If not, the app moves it to `Ignored`.

#### 2. Extraction

The model gets text and images. So the app converts every part of the email to text or to an image:

- **The body.** The app takes the `text/plain` part of the body. When the email has only an HTML part, it converts the HTML to text. Long bodies are cut at 40,000 characters.
- **PDFs.** If the pages have a text layer, more than 200 characters per page on average, the app extracts the text. If not, it is probably a scan, and the app renders each page to a PNG image.
- **Photos.** JPEG, PNG and GIF attachments go to the model encoded in base64. Images under 20 KB (logos, signatures, etc.) are skipped.

Attachments over 10 MB and files of other types do not go to the model.

#### 3. Prompt

The app assembles the request in the Anthropic Messages API format:

- The system prompt.
- One user message with a text block: the metadata (sender, subject, dates, etc.), the body text, the text of each PDF.
- The images, in the same user message, each as a base64 image block.

#### 4. Model call
The app sends the request to the DeepSeek Anthropic-compatible API. It has no JSON mode and no structured output, but it supports tools, and a tool has a JSON schema.

So, Instead of a text answer, the model calls one tool, `report_summary`, with the summary as the arguments of that call. The input schema of the tool is generated from a Pydantic model. The app validates the tool input against that model.

If the input is invalid or the model did not call the tool, the app sends the error back as a tool result and asks again. If the second answer is malformed too, the app posts the original text.

#### 5. Post

The app builds one Telegram message from the fields and sends it with the Bot API:

- A header line: the urgency, the category, the deadline and the event date.
- The title in bold.
- The summary.
- "What to do", when the model found an action.
- The attachment names, when there are any.
- A footer: the subject, the sender and the sent date of the email.

A Telegram message holds at most 4096 characters. A summary never gets close. The fallback post with the original text can, so the app cuts the text to fit.

#### 6. Move

The app moves the email to `Processed` with the IMAP MOVE command. From this moment the email counts as done.

### Post examples

```text
🔴 High · 🔄 Изменение расписания · 📅 Событие: пн, 12 окт
Понедельник 12.10 — выходной: День культурного разнообразия

В понедельник 12 октября детский сад не работает: государственный праздник —
Día del Respeto a la Diversidad Cultural (День уважения к культурному разнообразию).
Занятия возобновятся во вторник 13 октября. От родителей ничего не требуется.
📎 Вложения: Screenshot_20261009_114802_ChatGPT (1).jpg

Тема: DÍA DEL RESPETO A LA DIVERSIDAD CULTURAL · От: saladedosceleste2026@gmail.com
Отправлено: сб, 10 окт 2026 11:00
```

The first line shows what kind of email it was: urgency, category, deadline, event date. Then the title and the summary itself: what, when, what to do or bring. When there is something to do, it gets its own line. The attachments are listed by name but not uploaded. 

A couple of real examples:

<figure class="half">
  <a href="/images/kindergarten-email-checker-email-example_01_em.png"><img src="/images/kindergarten-email-checker-email-example_01_em.png" alt="Email: Día del respeto a la diversidad cultural"></a>
  <a href="/images/kindergarten-email-checker-email-example_01_tg.png"><img src="/images/kindergarten-email-checker-email-example_01_tg.png" alt="Post: Monday 12.10 is a day off"></a>
</figure>
---
<figure class="half">
  <a href="/images/kindergarten-email-checker-email-example_02_em.png"><img src="/images/kindergarten-email-checker-email-example_02_em.png" alt="Email: Biblioteca circulante"></a>
  <a href="/images/kindergarten-email-checker-email-example_02_tg.png"><img src="/images/kindergarten-email-checker-email-example_02_tg.png" alt="Post: Biblioteca circulante project"></a>
</figure>

## Interesting details

### Gmail labels as the database

There is no database and the tool has nothing on disk. An email is in exactly one label, and the label says what happened to it. 

![Gmail labels as the state](/images/kindergarten-email-checker-labels.png)

Bonus point: the Gmail web UI is the admin interface. I open the `Failed` label and see what went wrong. I drag an email back to `INBOX` and the tool processes it again at the next check.

### DeepSeek quirks

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

However DeepSeek pretends to be a cheap version of Antropic, it doesn't.
Two bugs that took a while to figure out.

1) **Thinking and a forced tool call do not work together.**
Thinking must be turned on: the emails have dates like "el viernes" that must be resolved from the sent date. Also urgency needs a judgement.
At the same time the answer must be as structured data, so the summary comes back as a call of the `report_summary` tool, whose JSON schema is generated from a Pydantic model. The Messages API has a way to force a specific tool: `tool_choice={"type": "tool", "name": "report_summary"}`. But DeepSeek rejects it with the error: `Thinking mode does not support this tool_choice`.
The workaround is `tool_choice` set to `auto`, plus a system prompt that tells the model to call the tool exactly once.

2) **Invalid JSON looks like an empty call.**
Occasionally DeepSeek writes tool arguments that are not valid JSON and doesn't report an error. It returns the tool call with the input `{}`. Pydantic rejects that, and the app sends the error back as a `tool_result` block with the full previous response and thinking blocks included. The second attempt has been fine every time so far. 

### "null" is not null

The schema has four optional fields: `action`, `deadline`, `event date` and `event time`. The model sometimes writes the string `"null"` instead of a JSON null. As a fix, a validator handles it: the strings `""`, `"null"` and `"none"` count as no value.

### Urgency is about time, not about the topic

The first prompt version said: `high` for anything urgent, cancelled, closed, or about health or safety. The model did exactly that. Every health notice became red, including "if your child has symptoms, keep them at home". This was a false alarm.

Now the prompt asks one question: how soon must the parent do something?

- `high`: act within two days, or attendance changes within two days.
- `medium`: act later, or an event with the child later.
- `low`: nothing is required. General rules and conditional instructions, such as "if your child has symptoms, keep them at home", are `low`.

The topic went to a separate field, the category: `payment`, `event`, `closure`, `health`, and so on. Category says what the email is about, urgency says how soon. So a health notice can be `health` and `low` at the same time. The category is only a label in the first line of the post. 

### Images and PDFs

The Anthropic API can take a PDF as a document block, and the model reads it. DeepSeek's endpoint does not support that. So the app does the transformations:

| Input | What the model gets |
|---|---|
| HTML body | The text/plain part if there is one, otherwise html2text |
| PDF with a text layer | The text, extracted with pypdfium2 |
| PDF without a text layer | Each page rendered at 110 dpi, at most 10 pages |
| JPEG, PNG, GIF, WebP | As is, when larger than 20 KB |
| Anything else | Not sent. The file name is listed in the prompt and in the post |


### Dates and the timezone

The prompt gives the model the sent date, the weekday and today's date, all in the Buenos Aires timezone. The emails say "el lunes" or "el viernes", not a date, and the model can only resolve that from the day the email was sent. The first example above is one: the text says "el día lunes", the Telegram post says 12 October. 

### The bot token in the logs

The Anthropic SDK and the tool itself use `httpx2`, and at the `INFO` level it logs the URL of every request. Telegram URLs contain the bot token. A log formatter replaces the token in every line, tracebacks included. The path reads `/bot<token>/`.

## Three weeks in Live

So far, the app has being running for thee weeks on the free k3s cluster from the [previous post](https://evgeniy.tech/2026/10/20/oracle-cloud-k3s.html). 
The Dockerfile, the Helm chart and the GitHub Actions pipeline are in the repository.

[SCREENSHOT: kubectl get pods and the logs of the app]

Here is some statistic.

- [NUMBER: emails processed]
- [NUMBER: posts without a summary, the fallback]
- [NUMBER: emails in Failed]
- DeepSeek cost: [NUMBER] for the period. The estimate was "cents per month", and it holds.

[SCREENSHOT: an example where the summary got a tricky email right, for example a relative date or a scanned PDF]

The code is on GitHub: [DeV1L/kindergarten-email-checker](https://github.com/DeV1L/kindergarten-email-checker).

## What is next

I have about 100 emails from the kindergarten by now, and the app has processed only a part of them. I'm going to build a golden dataset and can run different models against it and measure them: accuracy on each field, cost per email, speed, etc. 

That probably will be the next post. Stay tuned.


