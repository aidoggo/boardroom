# Boardroom

Boardroom is a Claude Code plugin that puts two or more of your AI agents in one meeting with you. Each agent comes to the table with its own rules, files and voice, and the agents answer each other directly. You stop carrying messages between them, and you make every decision yourself, one question at a time.

## Who it's for

Boardroom is for anyone who runs more than one Claude agent or persona and wants them to talk to each other. You might keep a legal agent in one folder and a finance agent in another, each with its own instructions and records. When a decision touches both, you usually ask one, copy its answer to the other, and copy the reply back. Boardroom replaces that relay with a meeting.

## What it needs

- Claude Code, in the terminal, an IDE extension, or the desktop app's Code tab. Boardroom launches background agents with the Agent tool and continues each one with the `SendMessage` tool. Claude.ai chat does not run agents, and Cowork's documentation does not describe continuing an agent with `SendMessage`, so Boardroom is built and tested for Claude Code only.
- Two or more agents that each have a home folder: a directory holding the instructions and files that agent reads before it speaks, such as a `CLAUDE.md`, a decisions file and its working records.
- Optionally, a connected calendar tool, if you want your own action items placed on your calendar at the close.

## Install

Add the marketplace and install the plugin from your shell:

```bash
claude plugin marketplace add aidoggo/boardroom
claude plugin install boardroom@boardroom
```

Start a new Claude Code session, or run `/reload-plugins` in an open one. The skill is available as `/boardroom:boardroom`, and Claude also starts it when you ask to convene your agents or hold a boardroom meeting.

## Set up your members

Boardroom reads your members from a roster file. The plugin itself installs into a read-only cache, so the roster lives in one of two places that you own:

- `~/.claude/boardroom/members.md` is your roster for every project.
- `.claude/boardroom/members.md` inside a project is that project's roster. When both files exist, the project roster wins and replaces yours entirely, settings included.

The first time you start a meeting without a roster, Boardroom runs a short setup. It asks one question at a time: where to save the roster, and then, for each member, its name, its home folder, the files it loads, and what it speaks on. It checks that the folders and files exist and writes the roster for you. Claude Code treats files under `.claude/` as protected, so it asks you to approve that write.

You can also write the roster by hand. The plugin ships a template, `skills/boardroom/members-template.md`, with three fictional members (a contracts lawyer agent, a finance agent and a product agent) that show you how each is structured. Each member section looks like this:

```markdown
## finance

- **Who:** a finance agent that keeps the company's books and forecasts.
- **Home:** `~/agents/finance`
- **Loads:** `CLAUDE.md`, `decisions.md`, `references/accounts.md`.
- **Speaks on:** revenue, costs, cash, runway, pricing, and what a decision costs.
- **Files outcomes:** a dated entry in `rulings.md`.
```

Every member receives the same seating prompt, built into the skill, which tells it the question, the other members and the room rules. To change it, add a `## Seating prompt` section to your roster; the template has a copy to start from. The roster also has a settings section:

| Setting | Default | What it does |
| :- | :- | :- |
| Transcript folder | `~/boardroom/` | Where meeting transcripts are written. It must be outside every git repository. |
| Your name | the owner | What members call you. |
| Time zone | not set | Used only by the calendar step. Boardroom asks for it the first time that step runs and saves your answer. |
| Calendar block time | `09:00` | The local time for task blocks on your calendar. |

If an agent is defined as a Claude Code subagent file rather than a folder, list that file under **Loads** and use the folder it sits in as the **Home**.

## A meeting from start to finish

1. **You ask for a meeting.** For example: "Convene the boardroom on whether we should accept the reseller's contract terms." The session you ask becomes the chair. It is neutral: it never argues a position and never speaks for a member. For that reason it works best when you open it from a folder that belongs to no member.
2. **The chair confirms the question and the attendees** in one line, names the roster it is using, and creates a transcript in your transcript folder.
3. **Each member is seated.** The chair launches one background agent per member, all at once, each with the seating prompt, the question and your framing. Each agent reads its own files first. The chair keeps every agent for the whole meeting and continues it with `SendMessage`, so each member remembers everything said so far.
4. **Opening round.** Every opening statement goes into the transcript word for word. You see a short card of each member's priorities, attributed to that member, and the path to the full text.
5. **Discussion.** The chair names where members disagree or have asked each other something, and routes each question to the member it is addressed to. Every message to a member carries, word for word, every turn since that member last spoke, including yours, so everyone hears the same room. Turns run about 250 words.
6. **You decide, one question at a time.** When a question needs your decision, the chair puts exactly one question to you, with a card showing each member's points and recommendation in that member's own words, and a neutral statement of the choice. If more questions are waiting, it tells you only how many. You give a provisional answer, name who you want to hear from, hear their concerns, and then decide. If the app you are using can show an HTML page privately, such as in an artifact panel, the card is a small page; otherwise it is a compact text card in chat.
7. **Homework goes to the parking lot.** When a question depends on facts nobody in the room has, such as a receipt, an exact amount or an account login, the chair does not send you to find it. It records the item in a parking lot, names the member who will dig it out after the meeting, and moves on.
8. **Close.** The chair writes the minutes into the transcript: every decision recorded as yours, action items with an owner each, the parking lot, and open questions. Each member then files its own outcome in its own home, following its own rules, and reports the paths it wrote. If a calendar tool is connected, each action item that needs you becomes a short block on your calendar before its due date.

## What it never does

- It never makes a decision for you. Members advise; the minutes record every decision as yours.
- It never paraphrases a member in place of showing its words. The transcript holds every turn verbatim, and every recommendation on a card is attributed.
- It never puts more than one question to you at a time, and it never hands you homework in the middle of a meeting.
- During the meeting, no member writes to any file or repository, sends email, creates drafts or calendar events, or spends money. Members may read their own files and run their own read-only tools.
- No member writes in another member's home. At the close, each member files only its own side.
- It never writes the transcript inside a git repository, because a transcript holds every member's material. If your transcript folder is inside a repository, the chair stops and asks for another folder.

## What it reads and writes

Boardroom contains no code and makes no network requests of its own. It is a set of instructions that Claude Code follows. During a meeting:

- Each member's agent reads the files listed for it in the roster, and may run that member's own read-only tools.
- The chair writes one transcript file per meeting in your transcript folder, which it creates with owner-only permissions.
- At the close, each member writes its outcome in its own home folder.
- Only if a calendar tool is connected, the chair creates calendar events for your action items.
- Only if the app can show an HTML page privately, the chair shows the decision cards there. It never posts them to a document, sharing or messaging service.
- During setup, the chair writes your roster file.

Background agents ask for permission in your main session whenever a tool call needs it, under your usual Claude Code permission settings.

## License

MIT. See [LICENSE](LICENSE). The Boardroom name and logo are covered separately by the [trademark policy](TRADEMARKS.md), which explains how to name a fork.

Landing page: [aidoggo.github.io/boardroom](https://aidoggo.github.io/boardroom/), built from [`index.html`](index.html).
