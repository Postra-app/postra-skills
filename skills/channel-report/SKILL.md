---
name: channel-report
description: Write a plain-words performance report for the social channels connected to Postra, with the numbers for a period and what changed. Use when the user asks how their channels or posts did, or wants a weekly or monthly social media report from Postra.
---

# Channel report

Needs the Postra MCP connector. If the Postra tools are missing, tell the user to connect it (see the README of this repository) and stop. This skill only reads. It changes nothing.

## Steps

1. **Pick the period:** what the user said, or the last 7 days. For a monthly report use 30.
2. **Call `integrationList`.** Report on the channels the user named, or on all of them. Names repeat across platforms, so always show the platform next to the name.
3. **Call `getAnalytics`** once per channel with its `integrationId` and `days`.
4. **Call `listScheduledPosts`** for the same period (`startDate`, `endDate` as full ISO date and time in UTC) to count what was published on each channel (state `PUBLISHED`) and what failed (`error` is set).
5. **Write the report:**
   - One line at the top: the single most important thing that happened.
   - One short table: channel, followers, the two or three numbers the platform gives (reach, engagement, views), posts published.
   - Two or three sentences on what changed and a likely reason, tied to what was posted.
   - Failed posts, with the reason.
   - One suggestion for next week.
6. **Be exact about what you do not know.** Platforms give different numbers, and some give none. If `getAnalytics` returns nothing for a channel, write "no analytics available for this channel" and do not estimate. Do not compare with an earlier period unless you fetched that period too.
