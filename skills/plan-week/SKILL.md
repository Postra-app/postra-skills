---
name: plan-week
description: Plan a week of social media posts in Postra from a short brief. Checks what is already in the calendar, writes a post for each channel in that platform's own style, and saves the week as drafts or schedules it. Use when the user asks to plan, fill or schedule a week (or several days) of posts with Postra.
---

# Plan a week in Postra

Needs the Postra MCP connector. If the Postra tools are missing, tell the user to connect it (see the README of this repository) and stop.

## Steps

1. **Get the brief.** You need: what the week is about (offers, news, themes), which channels, the user's timezone, and how many posts per channel. If the user gave no number, propose 3 per channel and say so. Ask only for what is missing, in one message.
2. **Call `integrationList`.** Match the channels the user named. No channels named: list them and ask which to use.
3. **Call `listScheduledPosts`** for the week (`startDate`, `endDate` in UTC). Note the days and times already taken on each channel, so the plan fills gaps and does not double up.
4. **Call `integrationSchema`** once per platform in the plan.
5. **Write the plan.** One table: day, local time, channel, post text. Rules for the writing:
   - Each channel gets its own version. Do not paste the same text everywhere: LinkedIn takes a few short paragraphs, X one or two sentences, Instagram a caption that works under a picture.
   - Spread the posts over the week. No two posts on one channel within 4 hours.
   - Concrete facts from the brief only. Do not invent prices, dates or claims.
   - Platforms that need a picture or video (Instagram, TikTok, Pinterest, YouTube): say which posts need media and ask whether to use a file from the library (`mediaListTool`), a link (`uploadFromUrlTool`) or an AI graphic (`createBrandedDraft`). A post for these platforms without media cannot be saved.
6. **Ask for a yes,** and whether to save as drafts or schedule.
7. **Call `integrationSchedulePostTool`** with one `socialPost` entry per post.
8. **Check.** Call `listScheduledPosts` for the same week and confirm every returned `postId` is there with the state you asked for (`DRAFT` or `QUEUE`). Report: how many were saved, as what (draft or scheduled), and anything that failed with its reason.

## Exact tool inputs

- `integrationList` returns each channel's `id`, name and platform. Names repeat across platforms, so match on name and platform together, and show the platform to the user.
- `integrationSchema` takes `platform` exactly as `integrationList` returns it (for example `linkedin-page`, `instagram-standalone`, `x`) and `isPremium` (`false` unless the user has X Premium).
- `integrationSchedulePostTool` takes `socialPost`, an array with one entry per post. Every entry needs all of:
  - `integrationId`: the channel `id`
  - `isPremium`: `false` unless the user has X Premium
  - `date`: full ISO date and time in UTC, for example `2026-11-17T10:00:00.000Z`
  - `shortLink`: `false` unless the user asks for short links
  - `type`: `"draft"`, `"schedule"` or `"now"`
  - `postsAndComments`: `[{ "content": "<p>...</p>", "attachments": [] }]`. The first item is the post, later items are comments or thread parts. `attachments` is required even when empty.
  - `settings`: an array of `{ "key": ..., "value": ... }` pairs, one for each setting the schema marks as required. Do not send `__type`.
- The call returns a `postId` for each post. It is the `id` that `listScheduledPosts` shows. Use it to check the post was saved.
- `listScheduledPosts` returns one row per channel with `id`, `group`, `preview` (the first words), the date, `error` and a state: `DRAFT`, `QUEUE` (scheduled) or `PUBLISHED`.

## Rules for every Postra tool call

- **Drafts by default.** Create posts with `type: "draft"` unless the user said, in this conversation, to schedule or publish. Never use `type: "now"` unless the user asked to publish now, in those words.
- **Show before you save.** Show the full text for every channel and the dates, and wait for a yes, before calling `integrationSchedulePostTool`.
- **Channels come from `integrationList`.** Use the `id` it returns. Never guess an id. If the user names a channel that is not in the list, say so and stop.
- **Ask the platform first.** Call `integrationSchema` for each platform before writing for it. It gives the maximum length and the settings that platform needs. Stay inside them.
- **Times are UTC.** The tools take a full ISO date and time ending in `Z`. Ask for the user's timezone if you do not know it, convert, and show both the local time and UTC in your summary.
- **Text is HTML.** Wrap every line in `<p>`. Allowed tags: `h1`, `h2`, `h3`, `u`, `strong`, `li`, `ul`, `p`. Do not put `u` and `strong` on the same text.
- **Attachments must be in the media library.** Use a URL returned by `mediaListTool`, `uploadFromUrlTool`, `generateImageTool` or `createBrandedDraft`.
- **AI images use the plan's monthly AI allowance.** Say how many images you are about to make and wait for a yes.
- **Scheduled posts count towards the plan's monthly posts.** Drafts do not.
- **Moving and deleting only ask.** `reschedulePost` and `deletePost` do not change anything through MCP. Tell the user to do it in Postra, in the calendar.
- **If a tool returns an error,** read it, fix the input and call it again. If it fails twice, show the user the error as it is.
