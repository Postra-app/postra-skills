# Postra skills for Claude

Plan, write and schedule social media posts in [Postra](https://postra.co.uk) from Claude. Connect Postra once, then add the skills you want.

## Set up

1. **Connect Postra.** You need a paid Postra plan, and you need to be an owner or admin of the workspace.

   In Claude: open Settings, then Connectors, then Add custom connector. Name it Postra, paste the address below and sign in with your Postra account.

   ```
   https://app.postra.co.uk/api/mcp
   ```

   In Claude Code, use your API key from Postra (Settings, Developers, Access):

   ```bash
   claude mcp add postra \
     --transport http \
     --header "Authorization: Bearer YOUR_API_KEY" \
     "https://app.postra.co.uk/api/mcp"
   ```

   Other assistants: see [Connect your assistant](https://postra.co.uk/docs/mcp/setup/).

2. **Add a skill.** In Claude: zip a folder from `skills/`, then open Settings, Capabilities, Skills and upload it. In Claude Code: copy the folder into `~/.claude/skills/`.

3. **Ask.** Type `/` and pick a skill, or ask in your own words: "Plan next week on LinkedIn and Facebook around our spring sale."

## Skills

| Skill | What you get |
|---|---|
| `plan-week` | A week of posts from a short brief, written for each channel and fitted around what is already in your calendar |
| `blog-to-posts` | One article turned into a post for each channel, each from a different angle, with an optional branded graphic |
| `branded-post` | One finished post with a matching graphic in your brand colours and voice |
| `calendar-check` | What is planned, what failed and why, and which days are empty |
| `channel-report` | A plain-words report on how your channels did |

## Good to know

- **Nothing goes out without your yes.** Every skill shows you the posts first and saves them as drafts unless you ask to schedule.
- **Posts are created in Postra,** so you can open, edit and move them in the calendar like any other post.
- **Plan allowances apply.** Scheduled posts count towards your plan's monthly posts. Drafts do not. AI graphics use your plan's monthly AI allowance, and the skill tells you before it makes one.
- **Moving and deleting posts is done in Postra.** Through the connector an assistant can only ask.
- The full list of tools is in the [Postra MCP documentation](https://postra.co.uk/docs/mcp/tools/).

## License

MIT. See [LICENSE](LICENSE).
