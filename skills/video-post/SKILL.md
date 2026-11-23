---
name: video-post
description: Make a short AI video from a description, or use a video the user already has, and post it with Postra with the right cover and settings for each platform. Use when the user asks for a video post, a reel, a short, a TikTok, or an AI-generated video for their channels with Postra.
---

# A video post

Needs the Postra MCP connector. If the Postra tools are missing, tell the user to connect it (see the README of this repository) and stop.

## Steps

1. **Find out where the video comes from:**
   - **The user has one:** a file already in the library (`mediaListTool`, search by file name) or a public link (`uploadFromUrlTool`). A file on their computer cannot be reached from here: tell them to upload it in Postra, in Media, and then find it with `mediaListTool`.
   - **Make one with AI:** go to step 2. Otherwise go to step 5.
2. **Call `generateVideoOptions`.** It lists the video types the workspace can make and the inputs each one takes. Use only what it returns.
3. **Agree the video with the user:** what happens in it, in two or three sentences, and the shape: `vertical` for reels, shorts and TikTok, `horizontal` for YouTube and feeds. Up to three reference pictures from the media library can be passed if the type accepts `images`.
   **Say that this uses their monthly AI video allowance, and wait for a yes.**
4. **Call `generateVideoTool` once** with the `identifier` from step 2, the `output` shape and the `customParams` that type needs (for example `prompt`). It only starts the video. Tell the user it takes a few minutes, that Postra will notify them, and that it will appear in their media library. Do not call the tool again for the same video and do not wait in a loop: when the user comes back, find the video with `mediaListTool`.
5. **Pick the cover.** Ask whether to use a picture as the cover (from the library, a link, or an AI graphic). No picture: the platform chooses a frame.
6. **Call `integrationList`,** then **`integrationSchema`** for each platform. Read its `rules`: which platforms need a title, a cover as a second attachment, a privacy setting or a post type. Fill settings that need an id with `triggerTool`.
7. **Write the caption** for each platform inside its limit, and a title where the schema asks for one.
8. **Show before saving:** the video link, the cover, the caption and title per channel, and the time in local time and UTC. Wait for a yes, and ask whether to save as drafts or schedule.
9. **Call `integrationSchedulePostTool`** with one entry per channel. The video goes in `attachments`, with the cover as described under "Video" in the rules below.
10. **Check** with `listScheduledPosts`. Tell the user that video platforms take a few minutes to process a video after it is published, so a link can open a little later than the post's time.

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
