# Postra skills for AI assistants

Plan, write and schedule social media posts in [Postra](https://postra.co.uk) from your AI assistant. Connect Postra once, then add the skills.

The skills are packaged for Claude and were tested with Claude. Each one is a plain text file, so other assistants can use them too: see [Other assistants](#other-assistants).

## Set up

### 1. Connect Postra

You need a paid Postra plan, and you need to be an owner or admin of the workspace.

**Claude (web and desktop):** open Settings, then Connectors, then Add custom connector. Name it Postra, paste the address below and sign in with your Postra account.

```
https://app.postra.co.uk/api/mcp
```

**Claude Code:** use your API key from Postra (Settings, Developers, Access).

```bash
claude mcp add postra \
  --transport http \
  --header "Authorization: Bearer YOUR_API_KEY" \
  "https://app.postra.co.uk/api/mcp"
```

Other assistants: see [Connect your assistant](https://postra.co.uk/docs/mcp/setup/).

### 2. Add the skills

**Claude Code:** two commands install all of them.

```
/plugin marketplace add Postra-app/postra-skills
/plugin install postra-skills@postra-skills
```

To get new versions later: `/plugin marketplace update postra-skills`, then update the plugin from `/plugin`.

**Claude (web and desktop):** download this repository, zip the folder of the skill you want from `skills/` (the folder itself, not only the file inside it), then open Customize, Skills, and upload the zip. Repeat for each skill you want.

### 3. Ask

Ask in your own words, or type `/` and pick a skill. In Claude Code the skills are listed as `/postra-skills:<name>`.

## Skills

| Skill | What you get | Try saying |
|---|---|---|
| `daily-post` | One piece of news on every channel: a long and a short version, one picture or video, the link in a first comment | "Post today's news on all my channels at 9: we now open on Sundays." |
| `plan-week` | A week of posts from a short brief, written for each channel and fitted around what is already in your calendar | "Plan next week on LinkedIn and Facebook around our spring sale." |
| `blog-to-posts` | One article turned into a post for each channel, each from a different angle, with an optional branded graphic | "Turn this article into posts for LinkedIn, X and Instagram: https://..." |
| `branded-post` | One finished post with a matching graphic in your brand colours and voice | "Make an Instagram post with a graphic: 20% off all bikes this weekend." |
| `video-post` | A short AI video from a description, or your own video, posted with the right cover and settings for each platform | "Make a 9:16 video of a barista pouring latte art and post it as a reel." |
| `repurpose-post` | A post you already published, turned into new posts for other channels or a later date | "Reuse last Tuesday's LinkedIn post on X and Threads." |
| `calendar-check` | What is planned, what failed and why, and which days are empty | "What is scheduled for next week, and did anything fail?" |
| `fix-failed-posts` | Each failed post explained in plain words, with a corrected post ready to go | "My Pinterest post did not go out. Fix it." |
| `channel-report` | A plain-words report on how your channels did | "How did my channels do in the last 30 days?" |

## Good to know

- **Nothing goes out without your yes.** Every skill shows you the posts first and saves them as drafts unless you ask to schedule.
- **Posts are created in Postra,** so you can open, edit and move them in the calendar like any other post.
- **Plan allowances apply.** Scheduled posts count towards your plan's monthly posts. Drafts do not. AI graphics and AI video use your plan's monthly AI allowance, and the skill tells you before it makes one.
- **Moving and deleting a post needs your click.** The assistant shows a confirmation card and the post changes only when you approve it. If your assistant shows no card, move or delete the post in Postra, in the calendar.
- The full list of tools is in the [Postra MCP documentation](https://postra.co.uk/docs/mcp/tools/).

## If something does not work

| What you see | What to do |
|---|---|
| The assistant says it has no Postra tools | The connector is not connected in this chat. Connect it (step 1) and start a new conversation. |
| "Unauthorized" or 401 | In Claude Code, the API key is wrong or was replaced: copy it again from Postra (Settings, Developers, Access). In Claude, remove the connector and sign in again. |
| A message about your plan or role | The connector needs a paid plan, and your account has to be an owner or admin of the workspace. |
| A channel is missing from the list | It is not connected in Postra, or your plan does not include that platform. Check Postra, Channels. |
| A post failed | Ask "what failed and why?" (`calendar-check`) or "fix it" (`fix-failed-posts`). If the reason is that the channel needs reconnecting, do that in Postra, Channels. |
| No confirmation card when moving or deleting | Your assistant does not show cards. Do it in Postra, in the calendar. |

## Other assistants

A skill is one Markdown file, `skills/<name>/SKILL.md`. Any assistant that is connected to Postra can follow it.

- **ChatGPT:** add Postra as a connector ([how](https://postra.co.uk/docs/mcp/setup/#chatgpt-and-claude-in-the-browser)), then paste the contents of the `SKILL.md` you want into the instructions of a project, or into the chat before your request.
- **Cursor, Codex, VS Code and other coding assistants:** connect Postra ([setup for each tool](https://postra.co.uk/docs/mcp/setup/)), then add the `SKILL.md` as a rule or instruction file, or point the assistant at it.

Only the lines at the top of the file between `---` are specific to Claude. The rest is ordinary instructions.

## Help and feedback

- Questions about your Postra account: [hello@postra.co.uk](mailto:hello@postra.co.uk).
- A skill that does the wrong thing, or an idea for a new one: open an issue in this repository.
- Postra itself is open source: [Postra-app/Postra-app](https://github.com/Postra-app/Postra-app).

## License

MIT. See [LICENSE](LICENSE).
