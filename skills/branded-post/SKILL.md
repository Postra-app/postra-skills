---
name: branded-post
description: Make one finished social media post in Postra with a matching graphic in the brand's colours and voice, saved as a draft or scheduled. Use when the user asks for a post with a graphic, a branded post, or an announcement or offer post with Postra.
---

# One branded post with a graphic

Needs the Postra MCP connector. If the Postra tools are missing, tell the user to connect it (see the README of this repository) and stop.

## Steps

1. **Get the subject:** what the post announces or offers, the one fact that matters (price, date, place), and the channel. Ask only for what is missing.
2. **Call `integrationList`** and find the channel.
3. **Tell the user this makes one AI graphic** from their monthly AI allowance, and wait for a yes.
4. **Call `createBrandedDraft`** with:
   - `topic`: the subject and the key fact in one or two sentences. This is also what goes on the image, so keep it concrete.
   - `platform`: the channel's platform, for example `instagram`, `facebook`, `linkedin`, `x`.
   - `language`: the language the user writes in, for example `English`.
   It returns `copy` (the caption), `path` (the graphic) and `headline`.
5. **Show the caption and the graphic link.** Ask whether to change the caption. The graphic itself can be edited in Postra, in Studio: it is saved in the media library with its design.
6. **Call `integrationSchema`** for the platform and check the caption fits.
7. **Call `integrationSchedulePostTool`** with the caption as the post and `path` as the attachment.
8. **Report** what was saved and where to find it in Postra.

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
