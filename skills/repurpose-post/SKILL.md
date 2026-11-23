---
name: repurpose-post
description: Take a post that is already published in Postra and turn it into new posts for other channels or a later date, each from a different angle. Use when the user wants to reuse, repost, recycle or follow up a post that did well, with Postra.
---

# Repurpose a published post

Needs the Postra MCP connector. If the Postra tools are missing, tell the user to connect it (see the README of this repository) and stop.

## Steps

1. **Find the post.** Call `listScheduledPosts` for the period the user means (the last 30 days if they did not say) and list the `PUBLISHED` posts: date, channel, first words and link. Ask which one to reuse.
   Postra's analytics are per channel and per day, not per post. If the user asks for "the best one", call `getAnalytics` for the channel, name the days with the highest engagement, and show the posts published on those days. Say that this points at a day, not at a single post.
2. **Get the full text.** `listScheduledPosts` returns only the first words. Ask the user to paste the post, or open its link if you can read web pages.
3. **Agree where the new posts go:** which channels (call `integrationList`), and when. A reused post works best at least a week after the original on the same channel.
4. **Call `integrationSchema`** for each platform.
5. **Write each new post from a different angle,** not a copy: the result, the how, a question to the reader, a short quote, a number. Each channel gets its own version inside its limit. Keep the facts of the original. Do not add claims it did not make.
6. **Media.** Reuse the original picture or video from the library (`mediaListTool`), or offer a new graphic (`createBrandedDraft`, after telling the user it uses their AI allowance).
7. **Show the posts and dates and wait for a yes.** Ask whether to save as drafts or schedule.
8. **Call `integrationSchedulePostTool`,** then check with `listScheduledPosts`.

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
- `listScheduledPosts` returns one row per channel with `id`, `group`, `channel`, `platform`, `preview` (the first words, not the whole text), the date, `url` once published, `error`, and a state: `DRAFT`, `QUEUE` (scheduled), `PUBLISHED` or `ERROR` (failed, with the reason in `error`).

## Rules for every Postra tool call

- **Drafts by default.** Create posts with `type: "draft"` unless the user said, in this conversation, to schedule or publish. Never use `type: "now"` unless the user asked to publish now, in those words.
- **Show before you save.** Show the full text for every channel and the dates, and wait for a yes, before calling `integrationSchedulePostTool`.
- **Channels come from `integrationList`.** Use the `id` it returns. Never guess an id. If the user names a channel that is not in the list, say so and stop.
- **Ask the platform first.** Call `integrationSchema` for each platform before writing for it. It gives the maximum length and the settings that platform needs. Stay inside them.
- **Times are UTC.** The tools take a full ISO date and time ending in `Z`. Ask for the user's timezone if you do not know it, convert, and show both the local time and UTC in your summary.
- **Text is HTML.** Wrap every line in `<p>`. Allowed tags: `h1`, `h2`, `h3`, `u`, `strong`, `li`, `ul`, `p`. Do not put `u` and `strong` on the same text.
- **Ids in settings come from the channel, never from a guess.** When `integrationSchema` lists a tool under `tools` (for example `boards` for Pinterest), call `triggerTool` with the channel `id` and that `methodName`, show the user the names it returns, and use the `id` of the one they pick. A name is not an id.
- **Video.** Attach the video's URL from the media library. To choose the cover, pass the attachment as `{ "path": "<video URL>", "thumbnail": "<image URL>" }`. Some platforms ask for more, and `integrationSchema` says so in `rules`: Pinterest, for one, needs the cover picture as a second attachment after the video.
- **A link works best in a first comment.** To add one, put a second item in `postsAndComments`: `{ "content": "<p>https://...</p>", "attachments": [] }`. Not every platform takes comments (Pinterest does not): leave the second item out there and put the link in the post's `link` setting if the schema has one.
- **Attachments must be in the media library.** Use a URL returned by `mediaListTool`, `uploadFromUrlTool`, `generateImageTool` or `createBrandedDraft`.
- **AI images use the plan's monthly AI allowance.** Say how many images you are about to make and wait for a yes.
- **Scheduled posts count towards the plan's monthly posts.** Drafts do not.
- **Moving and deleting need the user's click.** `reschedulePost` (takes the post `id`) and `deletePost` (takes the post `group`) change nothing by themselves: each shows the user a confirmation card, and the post moves or goes only when they approve it there. Call the tool once per request and point the user to the card. Do not ask for a "yes" in the chat instead. If the user says no card appeared (some assistants, such as terminal ones, do not show cards), tell them to move or delete the post in Postra, in the calendar.
- **If a tool returns an error,** read it, fix the input and call it again. If it fails twice, show the user the error as it is.
