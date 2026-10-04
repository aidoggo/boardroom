# Privacy policy

Boardroom is a Claude Code plugin made of instructions and a template. It contains no code, runs no servers, and makes no network requests of its own. The plugin's author receives no data from anyone who uses it.

## What it reads

During a meeting, each member's agent reads the files that you list for that member in your roster, on your own machine. During setup, the chair checks that those folders and files exist.

## What it stores

Boardroom writes only to your own machine:

- your roster, at `~/.claude/boardroom/members.md` or at `.claude/boardroom/members.md` in a project;
- one transcript per meeting, in the transcript folder you choose, which defaults to `~/boardroom/`;
- each member's outcome, in that member's own home folder, at the close of a meeting.

These files stay on your machine until you delete them.

## What it sends

Boardroom sends nothing to its author or to any service of its own. If you have already connected a calendar tool to Claude Code, the chair can create events there for your own action items at the close of a meeting. Those events go to the calendar service you connected, under that service's own terms.

Your conversation with Claude, including the meeting, is handled by Claude Code under Anthropic's terms and privacy policy, as it would be without the plugin.

## Questions

Open an issue at https://github.com/aidoggo/boardroom/issues.
