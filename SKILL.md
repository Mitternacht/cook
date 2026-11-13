---
name: cook
description: Manual only. Runs the Cook Loop for ambitious visual builds that must match reference images, screens or videos through long autonomous iteration. Start it only when the user types /cook (or $cook) and names the folder to work in and the subagent mode. Never start it from other phrases such as "go cook", "let it cook" or "match the references", and never on your own judgment.
disable-model-invocation: true
---

# Cook

Give your agent a target, a bar and time, then let it cook.

This skill runs the Cook Loop: one intake, then a long autonomous loop of build, render, compare, critique and fix until the work holds up next to the references. It is made for runs of 10 to 40 hours or more. The references carry the look. The brief sets the bar, the rules and the time.

Optional companions: /unlazy for effort and completion discipline (the Depth Tree), and any goal, loop or long-run mode the harness offers. Cook works without them.

## Fork rules (read first)

This fork adds three rules. They win over every other line in this skill.

1. **Manual start only.** Start a run only when the user types `/cook` (or `$cook`). Never start one from a phrase, an attachment or your own judgment. If you think cook would help, say so in one line and stop. Resuming a run the user already started, after a context reset, is fine.
2. **The user names the folder.** Work, write `cook/` and run commands only inside the folder the user named for this run. If none was named, ask for it at intake. Never assume the current directory and never touch files outside the folder.
3. **The user sets the subagent mode.**

| Mode | Allowed | Default |
|---|---|---|
| `none` | No subagents and no fresh sessions. You build and judge alone. | yes |
| `critics` | Only zero-context critics, 2 or 3 per round. You build alone. | |
| `many` | Parallel builders and critics, up to 12 at once. | |

If the user did not name a mode, use `none`. Never go above the mode, not even when behind on pace. Write the mode in `cook/BRIEF.md`. It changes only when the user says so in chat.

## State files

All run state lives in `cook/` at the root of the folder the user named. These files are your memory across context resets and the handoff to any other agent.

| Path | Holds |
|---|---|
| `cook/BRIEF.md` | Target, bar, rules, time, deliverable, permissions. Written once after intake. |
| `cook/SHOTS.md` | One shot per reference: camera, framing, what to take from it. |
| `cook/ROUNDS.md` | The ledger. Copy `templates/ROUNDS.md` and add one entry per round. |
| `cook/refs/` | The references, numbered R01, R02 for images and V01, V02 for videos. |
| `cook/stills/rNN/`, `cook/compare/rNN/` | Renders and side-by-side images of round NN. |

## Start or resume

- `cook/BRIEF.md` exists and the run is not shipped: you are resuming. Read BRIEF, SHOTS and the last two ROUNDS entries, then start the next round (step 6). Do not re-ask, restart or rebuild what works.
- After every context compaction, do the same before your next action.
- `cook/` holds a shipped run: move it to `cook-archive/<date>/` and start at step 1.
- No `cook/` folder: start at step 1.

## The Cook Loop

### 1. Intake (once)

Ask every open question in one message, each with a default, so the user can answer by number or reply "defaults". Skip what the prompt, the attachments or a filled `templates/brief.md` already answer. If nothing is open, say so and start. Questions and defaults: `references/intake.md`.

Collect: the folder and the subagent mode first (see Fork rules), then what to build, references, bar, time budget, hard rules, deliverable, hardware, usage limits, and what you may download or publish.

Then write `cook/BRIEF.md` and `cook/ROUNDS.md` and start. From now on, never ask a question. The user may be asleep. Decide, log the decision under Decisions in ROUNDS.md, and keep working.

### 2. Target

References are the minimum bar, not inspiration.
- Study every reference at full size before building anything.
- Write `cook/SHOTS.md`: one shot per reference image, with camera, framing, time of day and what to take from it (light, color, composition, materials, timing).
- Motion references: analyze them frame by frame into a list of effects, transitions and timings. Take from many, never copy one element 1:1.
- Getting references for 3D, UI and motion, and the analysis steps: `references/target-and-bar.md`.

### 3. Bar

Write a number per dimension into BRIEF.md: composition, light, color, materials, detail density (near and far), motion.
- Stills: a stretch target such as 99.9% similarity. Video: a score out of 10. References at about 8.5 mean a target of 9 or more.
- Models rate themselves too high. When the user gives an estimate, log the gap as the calibration offset and apply it to every later score.
- Keep measured numbers (a command produced them) apart from eyeballed ones (critic scores).

### 4. Rules

Defaults unless the brief says otherwise. Full rules: `references/rules.md`.
- Zero external assets: every mesh, texture, shader and effect in code, when the point is the flex.
- Sound: code-made only if it truly holds up. Otherwise licensed free libraries, credited.
- Launch videos: real logos and real photos beat drawn SVG people.
- Slop ban list: eyebrow labels, monospace uppercase text, random serif fonts, purple gradients, beige, 01/02/03 counters, glowing orbs, shockwave rings popping out of things, decorative particles, em dashes.
- Color matches the references: never grey or washed out, never oversaturated.
- Weak GPU: keep realtime cheap and get the quality from offline rendering (sub-frame accumulation for motion blur, soft shadows, depth of field, anti-aliasing).

### 5. Time

- Demos get a floor: at least 8h, and done after 2h means something is wrong. Big builds can take days. Small jobs get a hard cap.
- Log the start time. Read the clock at every round boundary.
- Pace usage: about 20% of a 5h window per hour, about 99% per window, never 100%.
- Depth over breadth: make one shot, scene or element exceptional before you expand.

### 6. Loop

Before round 1, build the harness: a fixed camera preset per shot, a still renderer and a color check. For side-by-side images use `scripts/side-by-side.mjs` from this skill (Node 18+ and ffmpeg): `node <skill>/scripts/side-by-side.mjs --out cook/compare/r01/all.jpg cook/refs/R01.png cook/stills/r01/R01.png ...`. Every round reuses the harness.

Each round:
1. Focus: open user feedback first, else the weakest dimension in the weakest shot.
2. Fix it in depth. In mode `many` only, split independent parts across parallel subagents. Otherwise build it yourself.
3. Render stills from the shot presets, same framing and aspect as each reference. For motion, render contact sheets and short clips.
4. Make side-by-side images: reference left, render right.
5. Measure what can be measured: color per region, frame time, seconds per rendered frame.
6. Score with fresh-context critics (modes `critics` and `many`): subagents with zero context that get only the side-by-side images and the rubric. Another model only if the user named one. In mode `none`, score it yourself and mark every score "self-scored". Your own score is logged, never decisive.
7. Log the round in `cook/ROUNDS.md`.
8. Send the stills to the user (Gate) and keep going.
9. Run the Stop check. If it fails, start the next round now.

Never stop when it first works. Never claim a check that was not run. If something was not measured, write "not measured". Round details, critic prompts and pacing: `references/loop-and-critics.md`.

### 7. Gate

- After every round, send the stills with a short note: what changed, scores, next focus. Do not pause, and do not end your turn to wait for a reply.
- User feedback beats critic priorities. Log the exact words and address them in the next round.
- Never start the final render, or any other long and costly export, before the user approves the stills in chat. While you wait, keep refining and test the render pipeline on a few short frames.

### 8. Ship

After approval, render the final deliverable and check the file. For a video on X: the first frame is the thumbnail and must be impressive, best scene first, a new scene every 6 to 7 seconds, no text on screen, licensed music credited, 1080p 30fps is enough. For a public repo: README, credits, the real prompt and honest numbers. Push or publish only if BRIEF.md allows it. Details: `references/ship.md`.

## Stop check

Stop only when all of these are true:
- Every dimension in every shot meets the bar, by critic median after calibration. In mode `none` there are no critics, so a self-scored bar never ends the run: stop at a hard cap, or when the user says it is done.
- The time floor is reached (capped jobs have no floor).
- The user approved the stills, and the final deliverable is rendered and its file checked.

Done before the floor means the bar was read too low. Reread the references at full size, check far distance, color and the weakest shot, and start another round. If a hard cap or a usage stop comes first, finish the round, log it and report plainly what is still below the bar.

## Never

- Ask a question after intake.
- Start the final render before the user approves the stills.
- Copy one reference element 1:1, or name third-party reference clips in public files.
- Let your own score decide a stop.
- Report a number you did not measure or a check you did not run.
- Use 100% of a usage window.
- Put API keys, tokens or private information in anything you publish.
- Start a run the user did not start with `/cook`.
- Touch anything outside the folder the user named.
- Use more subagents than the mode allows.

## Files in this skill

| File | Read when |
|---|---|
| `references/intake.md` | Step 1: questions, defaults, writing BRIEF.md. |
| `references/target-and-bar.md` | Steps 2 and 3: references, shot list, motion analysis, rubric, calibration, measuring. |
| `references/loop-and-critics.md` | Steps 6 and 7: round details, critic prompts, user feedback, pacing, long runs. |
| `references/rules.md` | Step 4, and whenever you add anything visible or audible. |
| `references/ship.md` | Step 8: final render, video spec, repo, honest numbers, privacy scan. |
| `templates/brief.md` | When a user wants to write the brief themselves. |
| `templates/ROUNDS.md` | At the start of a run: copy it to `cook/ROUNDS.md`. |
| `scripts/side-by-side.mjs` | Every round: reference left, render right, one row per shot. |
