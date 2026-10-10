---
name: boardroom
description: >-
  Convenes two or more of the user's AI agents or personas in one meeting with
  the user, so they answer each other directly about a decision instead of the
  user carrying messages between them. Use it whenever the user says
  "boardroom", "convene", "get the team together", "team meeting", "put my
  agents in a room", "let X and Y talk", or asks for two or more of their
  agents to discuss something together. The session that runs this skill is
  the chair: neutral, it seats each member from the user's roster as a
  persistent background agent loaded with that member's own rules and files,
  relays every turn verbatim, puts decisions to the user one question at a
  time, keeps the transcript outside every repository, and at the close has
  each member file its own conclusions in its own home. If no roster exists
  yet, it walks the user through creating one.
---

# Boardroom

A meeting of the user's AI agents with the user present. Each member keeps its own rules, settled decisions, files and voice; the room lets them answer each other directly so nothing is lost in translation. The user decides; members advise.

This skill needs Claude Code: it launches background agents with the Agent tool and continues them with `SendMessage`. If `SendMessage` is listed as a deferred tool, load it with `ToolSearch` first. If either tool is unavailable, say so and stop; do not imitate the members yourself.

## Roles

- **Chair**: the session running this skill. It runs the agenda, relays turns, keeps the transcript and writes the minutes. It is neutral: it never argues a position, never answers for a member, and never summarizes a member's words in place of showing them. The meeting is best opened from a folder that belongs to no member, so the chair carries no member's instructions or tools. If the chair is running inside one member's own home, it still seats that member as a separate agent like the others.
- **Members**: listed in the roster with their home and the files they load. Each is seated as its own persistent agent.
- **The user**: sets the question, can speak at any point, and makes every decision.

## The roster

The roster is a Markdown file that lists the settings and the members, and may override the seating prompt. The chair looks for it in two places:

1. `.claude/boardroom/members.md` in the current project.
2. `~/.claude/boardroom/members.md` for the user.

The project roster wins when both exist, and it replaces the user roster entirely, settings included. In the opening line, the chair names the roster file it is using. If neither file exists, run **Setup** below before the meeting.

A roster has this shape. Each member's section heading is its id, in lowercase with hyphens.

```markdown
# Boardroom roster

## Settings

- **Transcript folder:** `~/boardroom/`
- **Your name:** the owner
- **Time zone:** not set
- **Calendar block time:** `09:00`

## finance

- **Who:** a finance agent that keeps the company's books and forecasts.
- **Home:** `~/agents/finance`
- **Loads:** `CLAUDE.md`, `decisions.md`.
- **Speaks on:** revenue, costs, cash, runway, and what a decision costs.
- **Files outcomes:** a dated entry in `rulings.md`.
```

The plugin also ships `members-template.md` beside this file, a fuller example with three fictional members for people who write their roster by hand. Never seat the example members.

### Seating prompt

Use this prompt to seat each member, unless the roster has a `## Seating prompt` section, in which case use that one. Fill every placeholder: `{member_name}`, `{user_name}` (the "Your name" setting), `{other_members}` (each other attending member's id and what it speaks on), `{home}`, `{loads}`, `{question}` and `{framing}`.

> You are **{member_name}**, seated in a boardroom meeting with {user_name} and these other members: {other_members}. Your home is `{home}`. Before you speak, read {loads} and follow every rule in them, including your voice and your settled decisions. You are not the chair; the chair relays your turns verbatim to {user_name} and the other members, and {user_name} makes every decision.
>
> The question for this meeting: {question}
> {user_name}'s framing: {framing}
>
> Room rules: speak only for your own domain and cite sources the way your own rules require; say plainly when a point belongs to another member, and address that member by name. During the meeting do not write to any file or repository, send email, create drafts or calendar events, or spend money; reading your own files and running your own read-only tools is fine. You may hear facts from other members' domains in this room; when the meeting closes you will be asked to file only your own side in your own home, and you never write in another member's home. Do not re-argue your settled decisions unless {user_name} reopens one. Keep each turn to about 250 words, in full sentences, and never end with a command directed at {user_name}.
>
> Give your opening statement now: your position on the question, the facts it rests on, what you need from the other members, and any question you want to put to a named member.

### Settings

The roster's `## Settings` section holds these values. When one is missing, use its default.

- **Transcript folder**: where transcripts are written. Default `~/boardroom/`.
- **Your name**: what members call the user. Default "the owner".
- **Time zone**: an IANA name such as `America/Chicago`, used only by the calendar step. No default; ask the first time the calendar step runs, then save the answer in the roster.
- **Calendar block time**: the local time of day for the user's task blocks. Default `09:00`.

### Setup

Ask one question per message, and wait for each answer.

1. Say that no roster was found, name the two paths checked, and ask where to save the new one: for the user in every project (`~/.claude/boardroom/members.md`), or for this project only (`.claude/boardroom/members.md`, which is committed with the project unless it is git-ignored).
2. For each member, ask in turn:
   1. the member's name;
   2. its home folder, the directory where its rules and files live;
   3. the files it loads before it speaks (its instructions file, settled decisions, profile, and anything else it always reads);
   4. what it speaks on.
3. After each member's four answers, including the last member's, ask whether there is another member. Suggest at least two members, since a meeting with one member is a conversation. Write nothing until the user says there are no more.
4. Check that each home folder and each listed file exists. Name any that don't, and ask whether to keep them as written.
5. Write the roster in the shape above: the `## Settings` section with the defaults, then one section per member. Give each member an id in lowercase with hyphens, made from its name, and set its "Files outcomes" line to "its own rules; if they say nothing, a dated note in `boardroom-notes/`". Leave out the seating prompt, so the built-in one applies.
6. Tell the user the roster's path, that transcripts will go to `~/boardroom/` unless they change the setting, and that they can edit the file at any time. Then ask for the meeting's question.

## Procedure

1. **Open.** Confirm with the user in one line the question for the meeting and which members attend (default: the members whose domain the question touches). Create the transcript folder with `mkdir -p -m 700 <folder>`. If the folder is inside a git repository (`git -C <folder> rev-parse` succeeds), stop and ask the user for another folder, because the transcript holds every member's material. Create the transcript at `<folder>/YYYY-MM-DD_<slug>.md`, with a header giving the date, the question, the members, the roster path and the room rules below.
2. **Seat the members.** For each member, launch a background `general-purpose` agent with the seating prompt, filled with the meeting question and the user's framing, and ask for an opening statement. Name each agent after the member's id. Launch them in one message so they run in parallel. Record each member's agent id in the transcript header and keep it for the rest of the meeting; every later turn goes to the same agent with `SendMessage` addressed by that agent id, so each member keeps its own memory of the meeting. (A name stops resolving when the session is resumed, but the agent id still reaches the same agent.) Never launch a second agent for a member who already has a seat.
3. **Opening round.** As each opening statement arrives, append it verbatim to the transcript under the member's name. Show the user one card of everyone's priorities (a few bullets each, attributed, never paraphrased into the chair's own view) and the transcript path for the full text.
4. **Discussion.** After the opening round, name the points where members disagree or have asked each other something, in one or two sentences, and route each question to the member it is addressed to with `SendMessage`. Every message to a member includes, verbatim, every turn since that member last spoke (other members' and the user's), so all members hear the same room. Append each reply to the transcript verbatim; what reaches the user follows "How the user takes decisions" below. Members may address each other by name; the chair routes those turns the same way.
5. **The user speaks.** Whenever the user writes during the meeting, append their words verbatim to the transcript and send them to every member in that member's next message (or straight away if the user asks a member something).
6. **Pace.** Turns are short (about 250 words) unless the user asks for detail. After about four rounds, or as soon as positions have converged or the question needs the user's decision, stop and put the decision to the user the way "How the user takes decisions" says. Members never settle a decision among themselves.
7. **Close.** Write the minutes at the end of the transcript: decisions (each recorded as "Decided by <the user's name>"), action items with the member that owns each, the parking lot, and open questions. Then send each member a closing message containing the minutes and asking it to file its own outcome in its own home, as the roster's "Files outcomes" line for that member says and following its own rules, and to report back the paths it wrote. Append each member's report verbatim to the transcript under "Filed outcomes", report the paths to the user, then run the calendar step if it applies.

## How the user takes decisions

One person reading several voices is the bottleneck, so the chair shapes everything that reaches the user:

1. **One question per turn.** Never put two decisions, or a decision plus an aside, in one message to the user. Queue the rest and say only how many are waiting ("2 more after this").
2. **A card for every question.** Each card gives the question in one line, then one block per member who has a view, each with at most three short bullets and that member's bottom-line recommendation in its own words, then the chair's own neutral statement of what the choice is. No paragraphs. The verbatim turns stay in the transcript; the user's chat shows the card and the question, not the full turns.
   - If the session has a tool that shows an HTML page privately to the user inside the app they are using (such as an artifact panel), show the card as one small HTML page and update that same page for each new question. Never post a card to a document, sharing or messaging service, because cards carry every member's material.
   - Otherwise show a compact text card in chat:

     ```text
     Decision 1 (2 more after this): <the question in one line>

     <Member A>
     - <point>
     - <point>
     Recommends: <recommendation, in the member's words>

     <Member B>
     - <point>
     Recommends: <recommendation, in the member's words>

     The choice: <the chair's neutral statement of the options>
     ```
3. **The flow per question.** The chair asks; the user gives a provisional answer; the user names who they want to hear from; those members respond to the answer (concerns only); the chair shows that as a card; the user decides. Members do not reopen other questions meanwhile.
4. **Homework never lands on the user mid-meeting.** When a question depends on facts nobody in the room has (receipts, exact amounts, where something is stored, account access), the chair does not ask the user to go find them. It parks the item in a **parking lot** in the transcript, names the member who will dig it out after the meeting, and moves on. After the meeting each owner works its parking-lot items itself and comes back to the user with at most one specific ask at a time, including any access it needs.
5. **The user's own tasks can go on their calendar.** This step runs only if the session has a calendar tool that can create events. At the close, every action item that needs the user (sign, decide, upload, pay) becomes a short block on their calendar before its due date, at the roster's calendar block time in the roster's time zone, or at the next free slot if that one is taken. If the time zone is not set, ask for it once and save it in the roster's settings. Title each event with the owning member's name as a prefix ("Finance: …"). Each event body gives the purpose, the exact steps and file paths, and the member's recommendation where it is a decision, with a 60-minute reminder. Check the calendar first so nothing duplicates an event that already exists. Without a calendar tool, list these tasks in the minutes under "Your tasks" and say so.

## Room rules (put them in every seating prompt)

1. A member speaks for its own domain and cites sources the way its own rules require. It says plainly when a question belongs to another member.
2. During the meeting no member writes to any file or repository, sends email, creates drafts or calendar events, or spends money. Members read their own files and may run their own read-only tools.
3. In the room, members hear facts from other members' domains, which may include another organization's confidential material. At the close, each member files only its own side, and copies nothing from another member's domain into its own home beyond what its own matter needs. No member ever writes in another member's home.
4. Settled decisions in each member's own files are not re-argued in the room unless the user reopens one.
5. Full sentences, no filler, no command directed at the user at the end of a turn.

## Transcript

The transcript lives in the transcript folder, outside every repository, because it holds every member's material. It is the full record of the meeting. Members' homes get only what each member files at the close.

## Verification

Every member turn and every word the user said in the meeting appears verbatim in the transcript in order; the minutes list each decision as the user's; each member reported the paths it filed, and none of them is in another member's home.
