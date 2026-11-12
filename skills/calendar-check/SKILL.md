---
name: calendar-check
description: Review the Postra content calendar. Shows what is planned, which posts failed and why, and which days or channels are empty. Use when the user asks what is scheduled, whether anything failed, or what is missing in Postra for the coming days.
---

# Calendar check

Needs the Postra MCP connector. If the Postra tools are missing, tell the user to connect it (see the README of this repository) and stop. This skill only reads. It changes nothing.

## Steps

1. **Pick the period.** Use what the user said. No period given: the next 7 days. Ask for the user's timezone if you do not know it.
2. **Call `integrationList`** for the channel names. Names repeat across platforms, so always show the platform next to the name.
3. **Call `listScheduledPosts`** with `startDate` and `endDate` as full ISO date and time in UTC (ending in `Z`). Start from now and end at the end of the last local day of the period.
   It returns one row per channel with `preview` (the first words), the date, `error`, `url` once published, and a state: `DRAFT`, `QUEUE` (scheduled) or `PUBLISHED`. One post sent to several channels comes back as several rows: merge rows with the same date and `preview` into one line that lists the channels.
4. **Report, in this order:**
   - **Failed posts first.** For each: channel, when, the first words of the text, and the reason from `error` in plain words. If the reason says the channel needs reconnecting, say so: Postra, Channels, reconnect.
   - **Already published** in the period, with the link from `url`.
   - **Planned posts by day,** in the user's local time: time, channels, first words, and whether it is scheduled (`QUEUE`) or a draft.
   - **Gaps.** Channels with nothing planned in the period, and days with no posts at all.
   - **Clashes.** Two different posts on one channel within an hour of each other.
5. **Offer one next step,** for example planning the empty days. Do not create anything unless the user asks.

Moving and deleting do not work through MCP: `reschedulePost` and `deletePost` only ask. If the user wants a post moved or removed, tell them to do it in Postra, in the calendar.
