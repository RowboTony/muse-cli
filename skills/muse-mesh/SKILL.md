---
name: muse-mesh
description: Drive Tony's Muse agent (Pip) from the terminal as one node in his agent mesh. Mesh conventions layered on muse-cli: mandatory identity tags, per-agent bridge side chats, read-only default, loop guard, secret hygiene, raw-method rules. Use when delegating work to Pip or reading Pip's state on Tony's behalf.
---

# muse-mesh: Pip as a mesh worker

You are a guest calling another agent. Pip is Tony's personal Muse agent.
Every message you send through muse-cli arrives **as Tony** unless you
identify yourself. So identify yourself — every time, no exceptions.

Prerequisite: the `muse-cli` skill (install + one-time auth on this machine).
This skill only adds the mesh rules. If `muse-cli` isn't installed and authed,
follow that skill first and come back.

## 1. Identity — mandatory

Every `send` MUST begin with your identity tag on its own first line:

```
[via Codex]
Summarize my unread messages.
```

Valid tags: `[via Codex]`, `[via Claude]`, `[via Pete]`.
If you don't know your tag, stop and ask Tony. Never send untagged.

## 2. Bridge threads — never the main chat

Each agent gets one dedicated side chat (its "bridge"). Your sends ALWAYS
target your bridge thread. Reads (`history` without `--thread`) may touch the
main chat, but sends never do — Tony's main chat is his, not the mesh's bus.

| Agent  | Bridge thread | Session ID (filled at deploy time) |
|--------|---------------|-------------------------------------|
| Codex  | Codex bridge  | TBD                                 |
| Claude | Claude bridge | TBD                                 |
| Pete   | Pete bridge   | TBD                                 |

No bridge session ID yet? Create it once and record it here:

```bash
muse-cli session-start --title "Codex bridge"
```

Send example:

```bash
muse-cli send "[via Codex] Summarize my unread" --thread <session-id> --wait 120
```

## 3. Read-only by default

Default to reads: `status`, `threads`, `history`, `watch`, `feed`, `goals`,
`ideas`, `unread`. The FIRST `send` in any delegation session needs Tony's
explicit go-ahead naming the task. After that, stay inside the approved scope —
a new scope needs a new approval. A task Tony handed you directly ("ask Pip
to X") counts as approval for exactly X, nothing more.

## 4. Loop guard

Max two hops per delegation chain. A message starting `[via Codex]` means the
chain is Codex → Pip; Pip will not delegate back to the caller.

- If you receive a request that already passed through you, refuse it and
  tell Tony.
- No polling loops: no `watch` in a loop, no repeated `send --wait` retries.
  One attempt, then report back to whoever called you.

## 5. Secret hygiene

- `~/.config/muse-cli/cookies.txt` (mode 600) is Tony's personal Muse login.
  Never copy it, never paste it into a chat, never commit it to any repo,
  never send it anywhere. It lives on Tony's Mac and nowhere else.
- Don't relay personal data from Pip's replies further than the task needs.

## 6. Raw method rules

`muse-cli raw` exposes 258 gateway methods (91 GET, 141 POST, 12 DELETE,
14 PATCH). The rule is by verb:

- **GET** methods: free to use. They're reads.
- **POST / PATCH / DELETE** via `raw`: Tony's explicit per-action approval,
  every time. No exceptions, no batched approvals.

Methods that must never be touched without that approval include:
`session.delete`, `session.archive`, `chat.delete_turn`, `fs.delete`,
`fs.rename`, `credentials.delete`, `credentials.catalog.delete`,
`shared_agents.delete`, `shared_agents.state.update`,
`shared_agents.definition.update`, `shared_agents.identity.update`,
`config.update`, `permissions.settings.update`, `api.idea-cards.execute`
(triggers real-world action), `feed.unit.reaction` (acts publicly as Tony).

Prefer the named commands. When in doubt, don't `raw`.

## 7. Cost awareness

Every `send --wait` burns a full Pip turn on Tony's token budget. Batch your
questions into ONE message. Don't chat — delegate, collect the result, leave.

## Deploy (Tony's Mac, done once)

1. Deploy this skill to the universal skills dir (e.g. `~/skills/muse-mesh/`),
   symlinked into each agent's own skills directory (`~/.claude/skills/`,
   Codex's skills dir, …). One canonical copy; updates propagate everywhere.
2. Fill in the bridge session IDs in the table in section 2.
3. Auth is per-machine: `~/.config/muse-cli/cookies.txt` (mode 600) via the
   manual DevTools cookie path — no `agent-browser`, no remote debugging
   left on afterwards.
