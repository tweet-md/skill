# tweet.md Skill

Agent skill for using [tweet.md](https://tweet.md) to convert X/Twitter posts, threads, and articles into clean Markdown for LLMs, agents, RAG, research, and Obsidian.

The skill teaches compatible AI agents when and how to call tweet.md, including URL rewriting, API usage, authentication, thread handling, Obsidian output, and common error handling.

## Install

```bash
npx skills add tweet-md/skill
````

Or install directly from GitHub:

```bash
gh skill install tweet-md/skill
```

## Usage

After installation, ask your agent things like:

```txt
Read this X thread and summarize it:
https://x.com/jack/status/20
```

```txt
Convert this tweet to Markdown:
https://x.com/jack/status/20
```

```txt
Fetch this thread for my RAG pipeline:
https://x.com/jack/status/20
```

The agent should use tweet.md to return clean Markdown instead of messy copied text, screenshots, embeds, or HTML.

Put the link in the same message as the request, including queued or scheduled chat commands:

```txt
/queue use tweet-md to read https://x.com/jack/status/20
```

Replying to a message that contains the link only works when your agent passes the replied-to text into the turn.

### Hermes Agent: skill seems not to load

Hermes resolves skills by their frontmatter `name`. If two different copies named `tweet-md` sit in the same skills directory, for example `~/.hermes/skills/tweet-md` and `~/.hermes/skills/social-media/tweet-md`, Hermes treats the bare name as ambiguous:

* `skill_view("tweet-md")` is refused.
* No `/tweet-md` command is registered.
* `hermes skills list` shows only the categorized path. A copy at the top level can't be loaded by any name.

Keep a single copy. Afterwards, `hermes skills list` should show one `tweet-md` row.

On Telegram, Hermes includes the replied-to message's text in the turn, `/queue` included. If you quote only part of that message, only the quoted part is passed.

## What tweet.md supports

* Single X/Twitter posts
* Threads
* Quoted posts
* X Articles / long-form posts
* Profile fetching (bio, stats, pinned post, and a timeline of recent posts — replies and reposts optional)
* Markdown output
* Obsidian-friendly Markdown
* Browser URL rewriting from `x.com` or `twitter.com` to `tweet.md`
* Programmatic API usage with API keys

## Skill file

See [`SKILL.md`](./SKILL.md).

## Links

* Website: [https://tweet.md](https://tweet.md)
* Docs: [https://tweet.md/i/docs](https://tweet.md/i/docs)
* Credits: [https://tweet.md/i/topup](https://tweet.md/i/topup)
* API key dashboard: [https://tweet.md/i/dashboard](https://tweet.md/i/dashboard)

## License

MIT