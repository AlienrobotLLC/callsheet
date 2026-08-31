# Credits

Almost nothing here is original. This file says where each part came from, and what it taught
us — because "we found this obscure repo that already did it" is more useful to a reader than a
bibliography.

---

## The platform

Most of this is built on primitives that already existed in the coding-agent CLI, and reading
its documentation carefully changed the design more than once.

- **The hooks system** is the whole mechanism. Without a session-start, prompt-submit, pre-write
  and turn-end hook, none of this is possible — you'd be back to asking an agent to *remember*
  to coordinate, which it won't.
- **Cross-session messaging** is what the conversation tier rests on. A message starting a turn
  in an idle session is a real capability we did not have to build, and the fact that the vendor
  already **throttles inter-session message loops** (rate-limits repeats, caps the queue, so a
  loop terminates on its own) is a design lesson in itself.
- **The idle-notification subscription** independently states this project's central economic
  claim: it subscribes to a session's next idle moment *"without starting a turn or spending
  tokens in the watched session."* We built a cost ladder around that asymmetry; the vendor had
  already written it down. Cite it, don't claim it.
- **The agent-teams documentation** is refreshingly honest that file conflicts are handled by
  *convention*: "two teammates editing the same file leads to overwrites; break the work so each
  teammate owns a different set of files." That is advice, not a mechanism — and naming the gap
  that plainly is what made the claim protocol (§6) obviously worth building.
- **The agent-view panel** clarified the single sharpest distinction in this project. It shows
  exactly the status picture we wanted — working / needs input / idle — and the docs say plainly
  that *agent sessions cannot see or interact with it*. It is for the human. That one sentence
  is why everything here is delivered **to the agent**.
- A **public issue** on the CLI's tracker asking for a shared checkout guard and a session
  registry carried a measurement we would otherwise have learned by shipping a bug: over 30 days
  and ~13,800 write calls, **29% were correct absolute-path writes into other worktrees** that a
  gate keyed on the session's working directory would have judged wrongly. §6 keys on the
  write's target path because someone else counted.

## Projects that got here first

Found by going looking *after* building, which is the wrong order. All are worth your time.

| project | what it taught us |
|---|---|
| [banjoba/claude-coord](https://github.com/banjoba/claude-coord) | The closest match to the local design, feature for feature — session-start briefing, prompt-time injection, a pre-write hook that blocks on a peer's claim, an append-only log — and it does it **across two developers' machines**, syncing over a git branch instead of an API. If you want the no-server version of the shared board, start here. |
| [minchenlee/c9watch](https://github.com/minchenlee/c9watch) | Independently arrived at deriving status by matching processes to their transcripts — the same technique as §3 — and ships a **"needs attention"** state that separates *working* from *blocked waiting on a human*. That distinction is why §3 bothers to tell a long-running command apart from an unanswered permission prompt. |
| [leonletto/thrum](https://github.com/leonletto/thrum) | The most complete version of the hosted half: presence, cross-machine sync, phone notifications, and a **`who-has FILE`** query. That command is a straight borrow — "who is in this file right now" turns out to be the thing you want *before* you start work, not a warning at the write. |
| [louislva/claude-peers-mcp](https://github.com/louislva/claude-peers-mcp) | The most-used project in this space, and the reason to be careful with the word "peers". Lists running instances with a summary of what each is doing — as an MCP server rather than hooks, which is a legitimately different way to reach the same place. |
| [octoryn/octopus-blackboard](https://github.com/octoryn/octopus-blackboard) | Calls it a blackboard outright, and treats **decisions** as a first-class kind alongside claims and handoffs. We added a `decision` kind after reading it: "what did we settle, and why" is the entry people most often want to find again, and it was getting lost inside notes. |
| [Co-Messi/agent-peers-mcp](https://github.com/Co-Messi/agent-peers-mcp) | For stating the security posture bluntly: *peer messages are untrusted input and must never be treated as authority.* §11 exists because someone else said it first. |
| [PatilShreyas/claude-code-session-bridge](https://github.com/PatilShreyas/claude-code-session-bridge) | A useful foil rather than a borrow. It answers peer queries by having the listening agent respond from its live context — every question costs the peer a full turn. Seeing the alternative stated clearly is what made the cost ladder in §1 the first section rather than an afterthought. |
| [Untrivial-ai/agent-orchestrator](https://github.com/Untrivial-ai/agent-orchestrator) | The far better-known category — plan work, spawn a fleet, one worktree each, supervise from a board. Two things taken from it: **isolation beats claims when you control the spawn** (§14), and its Kanban derives state from live PR/CI/review facts rather than from what the agent says about itself. Same principle as §3, different evidence. |

**Amp** (Sourcegraph) deserves a separate mention: it ships `read_thread` / `find_thread`, letting
an agent search and read *other* threads without prompting their owner. That is the read-is-free
half of this project's argument, in a shipped product.

**Beads** (Steve Yegge) is adjacent and much larger: a distributed issue graph agents claim work
from, syncing across machines. Board-shaped state that crosses machines, arrived at from the
task-tracking direction.

## The literature

The idea is old, and the papers say it better.

- **Blackboard architecture.** Allen Newell coined the term in *"Some problems of basic
  organization in problem-solving programs"* (1962) — *"a set of workers, all looking at the same
  blackboard."* Herbert Simon proposed the idea to Raj Reddy and Lee Erman, and it was built as
  **Hearsay-II**: Erman, Hayes-Roth, Lesser & Reddy, *ACM Computing Surveys* 12(2), 1980. Its
  load-bearing sentence is the turn-end hook, forty years early: a knowledge source can use data
  *"without the creator of the data having to know which"* will use it. **H. Penny Nii's**
  *"Blackboard Systems"* (AI Magazine 7(2)/7(3), 1986) is the canonical explainer, and is careful
  to note that the control component is *not* specified by the model — which is the licence for
  §10 leaving it out.
- **Tuple spaces.** David Gelernter, *"Generative Communication in Linda,"* ACM TOPLAS 7(1),
  1985. A posted entry *"has an independent existence"* and is *"equally accessible to all
  processes, but bound to none"* — the reason a board entry needs neither the reader's identity
  nor the reader's existence. (Two details usually got wrong: the 1985 paper's primitives are
  `out`/`in`/`read`, not `rd`; and `eval` is described but not named there.) The
  space-and-time-decoupling vocabulary people attach to it is really **Eugster, Felber, Guerraoui
  & Kermarrec**, *"The Many Faces of Publish/Subscribe,"* ACM Computing Surveys 35(2), 2003.
  **Gelernter & Carriero**, *"Coordination Languages and their Significance"* (CACM 35(2), 1992)
  named the field.
- **Stigmergy.** Pierre-Paul Grassé, 1959, on termites: *"the stimulation of workers by the very
  performances they have achieved."* The distinction between **sematectonic** stigmergy (the work
  itself is the signal — auto-posted turn summaries) and **marker-based** (a dedicated signal —
  notes and claims) is **E. O. Wilson** (1975) and **H. Van Dyke Parunak** (E4MAS 2005)
  respectively. **Francis Heylighen's** *"Stigmergy as a Universal Coordination Mechanism"* (2016)
  is the best modern statement, and uses Wikipedia as its worked human example.
- **Coordination artifacts.** Omicini, Ricci, Viroli, Castelfranchi & Tummolini, *"Coordination
  Artifacts: Environment-Based Coordination for Intelligent Agents,"* AAMAS 2004. This is the
  closest formal statement of this project's thesis, twenty years early: direct interaction and
  explicit communication are not always the best route to coherent group behaviour. See also the
  **Agents & Artifacts** meta-model (Omicini, Ricci & Viroli, JAAMAS 17(3), 2008) and
  **CArtAgO** — a board is a *coordination artifact* in a precise, established sense.
- **Shared message pool.** Hong et al., *"MetaGPT,"* arXiv:2308.00352, ICLR 2024. Agents publish
  to a shared pool and subscribe by role, so any agent can retrieve what it needs *"eliminating
  the need to inquire."* The cost argument of §1, in a paper, before we had the problem. (They
  never call it a blackboard; that connection is ours, and it isn't much of one.)
- Worth reading if this interests you: **["Our AI Orchestration Frameworks Are Reinventing Linda
  (1985)"](https://otavio.cat/posts/ai-orchestration-reinventing-linda/)**, which makes half this
  argument better than we do.

## People

- A **development partner**, who has been on the other end of the shared board through every
  iteration and found more of its rough edges than any test did. Cross-developer coordination is
  not a design you can validate alone.
- One of the **agent sessions itself**, which — asked to run the self-check from its own session
  and report back — found that two of those checks were passing without testing anything (§12).
  The system reviewing its own work produced the two best paragraphs in that section, which is
  either a good sign or a joke about the whole enterprise, and possibly both.

## And the sets

The call sheet, department ownership, video village, and keeping the walkie for when you need
someone to *act* — a hundred people coordinating all day without interrupting each other. Film
crews solved this problem long before anyone had to solve it for software, and the answer was
never conversation.
