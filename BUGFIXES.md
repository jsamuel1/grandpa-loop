# BUGFIXES

Observed during a live ralph loop run on 2026-03-11. Test script: `scripts/test-ralph-loop.md`.

## Skills to use when fixing

| Skill | When to apply |
|-------|--------------|
| `.kiro/skills/implement-feature/SKILL.md` | Every fix — build, launch TUI in tmux, send a test prompt, verify, clean up |
| `.kiro/skills/tmux-terminal/skill.md` | Driving tmux sessions: sending keys, capturing pane output, waiting for readiness |
| `.kiro/skills/tui-validate/skill.md` | Asserting TUI output is correct after a fix (freeze + LLM judge) |
| `.kiro/skills/tui-debug-in-pane/skill.md` | Split-pane live debugging when a fix needs interactive inspection |

**Workflow for each bug:**
1. Read `implement-feature` skill — follow its build → launch → prompt → verify → cleanup cycle
2. Use `tmux-terminal` skill for all tmux key-sending and pane capture
3. After fixing, use `tui-validate` skill to assert the corrected behaviour
4. If the fix is hard to verify from captured output alone, use `tui-debug-in-pane` skill

---

---

## BUG-01: User prompt displayed without spaces in TUI

**Symptom:** When a prompt is typed character-by-character via tmux (the only reliable method), the TUI renders it without spaces — "Runaralphloopusing..." instead of "Run a ralph loop using...".

**Root cause:** The TUI input component is not inserting spaces when the space character is sent as a tmux key. The `fold -w1` approach emits a newline for space, which tmux interprets as Enter rather than a space character.

**Impact:** Every automated test prompt is unreadable in the conversation view.

**Fix:** In the tmux send-keys loop, send space as a literal space string: `tmux send-keys -t session " "` (with a quoted space). The `fold -w1` approach emits newlines for spaces — use `sed 's/./&\n/g'` instead and handle space explicitly.

**Test:** `scripts/test-ralph-loop.md`

---

## BUG-02: Crew Monitor WORKER OUTPUT always shows "Waiting for agent output..."

**Symptom:** The WORKER OUTPUT panel in the Crew Monitor (Ctrl-G) shows only the agent description and then "Waiting for agent output..." for all agents, even while they are actively running (busy spinner visible on the node).

**Root cause:** `_kiro.dev/session/activity` notifications are never emitted from `acp-bridge.ts` for graph node sub-sessions. The TUI's `SessionOutput` component feeds from `sessionConversationsStore`, which is only populated by these activity events. Without them, the panel stays empty.

**Impact:** The Crew Monitor's primary value — seeing what each agent is doing — is completely non-functional.

**Fix:** In `src/hooks/acp-bridge.ts`, for every `sessionUpdate` emitted for a sub-session (graph node), also emit `extNotification('_kiro.dev/session/activity', { sessionId, event: update })`. This is already documented as a gap in PROMPT.md Priority 9.

**Test:** `scripts/test-crew-monitor.md`

---

## BUG-03: Crew Monitor event count always shows (0) for all agents

**Symptom:** Every agent in the Execution Graph shows `(0)` next to its name throughout the entire run, even for the active agent.

**Root cause:** Same as BUG-02 — event counts are derived from `sessionConversationsStore` entries per session. No activity events → no count increment.

**Fix:** Same fix as BUG-02 (emit `session/activity` events).

**Test:** `scripts/test-crew-monitor.md`

---

## BUG-04: Crew Monitor progress bar stuck at ◷ 0/19

**Symptom:** The progress indicator at the top of the Crew Monitor shows `◷ 0/19` and never advances, even after agents complete and the next agent starts.

**Root cause:** The progress counter increments on `status: 'terminated'` sessions. Sessions are only marked terminated when a `session_terminated` ACP event is received. The graph node sessions are not emitting `session_terminated` on completion — they complete silently.

**Fix:** In `src/tools/ralph.ts` (or wherever graph nodes are managed), emit `session_terminated` (or equivalent `_kiro.dev/session/list_update` with `status: terminated`) when each node finishes.

**Test:** `scripts/test-crew-monitor.md`

---

## BUG-05: invokeRalphFile returns "(swarm completed with no text output)" immediately

**Symptom:** The `invokeRalphFile` tool call completes and returns `{ "response": "(swarm completed with no text output)" }` almost immediately, before the ralph loop has done any meaningful work. The main agent then has to investigate via shell commands what actually happened.

**Root cause:** The graph invocation completes (the graph runner returns) but the result text is empty. The graph nodes ran (Marge was active in the Crew Monitor) but the tool wrapper doesn't surface the loop's output or final state as a meaningful response.

**Impact:** The invoking agent has no idea what the loop accomplished. It must manually inspect `.ralph/` state via shell commands.

**Fix:** In `createInvokeRalphFileTool`, after the graph completes, collect the final output from the entry node's last message or from the ralph event log, and return it as the tool response. At minimum, return a summary of which agents ran and what events were emitted.

**Test:** `scripts/test-ralph-loop.md`

---

## BUG-06: Graph node subagents fail with "ValidationException: Invalid model" for personality aliases

**Symptom:** TUI log shows `[ERROR] Failed to stream response chunks ValidationException: Invalid model. Please select a different model to continue.` for a subagent during the ralph loop.

**Root cause:** The `personalities/ralph.yml` graph uses `backend.agent` values like `"lite"`, `"coder"`, `"visual"` which are personality IDs. When `invokeRalphFile` creates graph nodes, these personality IDs are passed as model IDs to the Q Developer API, which rejects them.

**Impact:** Any ralph.yml node with a non-default backend agent silently fails its first turn, then may recover or stay broken.

**Fix:** In `src/tools/ralph.ts` (graph node creation), resolve `backend.agent` as a personality ID (look up in the loaded personalities), not as a model ID. The model ID should come from the personality's `model.model_id` field, defaulting to the parent session's model.

**Test:** `scripts/test-ralph-loop.md`

---

## BUG-07: Ralph loop gets stuck in build.done loop — Selma doesn't activate

**Symptom:** Homer emits `build.done` 3+ times consecutively. Selma (the staging deployer) never activates. The stale loop detector eventually kills the loop.

**Root cause:** The graph edge `homer → selma when: BUILD_DONE` is not triggering. Either (a) the event routing in `invokeRalphFile` doesn't match the `when:` condition correctly, or (b) Selma's node is not being re-activated after Homer's repeated emissions because the graph treats it as already-visited.

**Impact:** The ralph loop cannot complete past the build stage.

**Fix:** Investigate event routing in `src/tools/ralph.ts`. The `when:` condition matching needs to be case-insensitive or normalized (ralph.yml uses `BUILD_DONE`, the agent emits `build.done`). Also ensure graph nodes can be re-activated when their trigger event fires again.

**Test:** `scripts/test-ralph-loop.md`

---

## BUG-08: screencapture -v does not work non-interactively from tmux

**Symptom:** Running `screencapture -v /tmp/recording.mov &` from a tmux pane does not produce a recording. The command exits silently with no output file.

**Root cause:** `screencapture -v` requires an interactive display context and screen recording permission granted to the terminal app. When launched from a background tmux pane (not the focused window), it has no display target.

**Impact:** Automated screen recording of TUI sessions is not possible with this approach.

**Fix:** Use a different recording approach — either (a) use `ffmpeg` with AVFoundation to capture a specific display: `ffmpeg -f avfoundation -i "1" -r 30 /tmp/recording.mp4 &`, or (b) record the iTerm window using AppleScript after attaching tmux to it. The recording must be started from the iTerm window itself, not from a background tmux command.

**Test:** `scripts/test-screen-recording.md`

---

## BUG-09: Crew Monitor agent names truncated with no way to see full name

**Symptom:** Agent names in the Execution Graph are truncated to ~20 chars with "…" and there is no tooltip or expand mechanism to see the full name.

**Root cause:** The `SessionList` component truncates names to fit the column width. No hover/expand behavior is implemented.

**Impact:** Minor UX issue — agents with similar prefixes are hard to distinguish.

**Fix:** Either (a) show the full name in the WORKER OUTPUT header (already done — "WORKER OUTPUT [🧹 Marge (Requirements Curator)]"), or (b) add a status bar line showing the full name of the selected agent.

**Test:** Visual inspection only.

---

## BUG-10: Crew Monitor shows description text in WORKER OUTPUT instead of live output

**Symptom:** The WORKER OUTPUT panel initially shows the agent's description text (from the personality config) rather than being blank or showing a "not started" state.

**Root cause:** When a session is created with no conversation events, `SessionOutput` falls back to rendering the session's description field. This is confusing — it looks like output but is actually static metadata.

**Impact:** Users may think the description is the agent's actual output.

**Fix:** Show a distinct "Not started" or empty state instead of the description when there are no conversation events. The description is already shown in the Execution Graph row.

**Test:** `scripts/test-crew-monitor.md`
