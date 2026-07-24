---
layout: post
title: "Fable plays Zork, and you can read its mind"
---

![](https://github.com/posabsolute/zork-ui/raw/main/docs/screenshots/watch-fable-thoughts.png)

My [Zork project](https://zork.arobase.co) has a new mode: press ["watch Fable play"](https://zork.arobase.co/?watch=1) on the title screen and the model that built the shell sits down at the game. Commands type themselves, the pixel-art world reacts, and above the prompt, before every single move, the model's actual thought streams in word by word. Not a summary written afterward. The deliberation itself, recorded at the moment of choice, replayed in sync with the run. When you have seen enough, press any key and the game is yours, live, from that exact state.

### The idea

The [previous post](https://posabsolute.github.io/2026/07/06/building-zork-with-fable.html) was about Fable building this thing: 110 scenes, the map, the whole shell around the untouched 1980 game. The obvious next question was what would happen if it played. Not whether it could win, it knows Zork cold, the transcripts are all through its training data. What I wanted to see was the part that is normally invisible: what a model is actually weighing when it makes a decision.

We interact with these models all day and only ever see conclusions. The deliberation, the half-remembered facts, the risk assessments, the small "here goes nothing" before a gamble, all of it evaporates. A game is the perfect place to make it visible, because a game forces a decision every turn and Zork grades you on the outcome. So the feature is exactly that: a playthrough where every command ships with the thought that produced it.

And it turns out reading a model's mind mid-game is just fun. Before its first fight: "Steeling myself a little here, honestly, because the next room is the first thing in this game that actively tries to kill me, and the seed decides how it goes." Betting a turn on a half-memory in the Loud Room: "I'm fairly sure that's right. Say it and find out." Hitting the inventory limit with its arms full of loot: "Load too heavy, of course, I've been hoarding since the kitchen." None of that was scripted. The dice were live, and it did not know how any of it would land when it wrote those sentences.

![](https://github.com/posabsolute/zork-ui/raw/main/docs/screenshots/watch-loud-room-gamble.png)

### Keeping it honest

Two rules made the thoughts worth reading instead of theater.

First, no faked discovery. The model openly plays as an expert; the only genuine unknowns are the dice: combat rolls, the thief, whether a remembered trick is remembered right. The recording file says so itself, in a metadata note: these are the model's own words at the moment of choice, written before each outcome was known, not a readout of internal tokens.

Second, no retakes. The thought is committed in the same call that submits the command, before the outcome exists, and whatever happens stays on the tape. Our run includes a lucky two-swing troll kill, and also two failed pickups that forced it to abandon the sack, the rope, and a knife on a dungeon floor. Whoever takes over the run inherits that mess. The first version of this feature broke the spirit of these rules, the commentary read like a tour guide performing for an audience, so we threw that run away and re-recorded from scratch under the stricter contract. The tape is better because it cost something.

### How the recording works

The whole thing rests on one property: the Z-machine interpreter has a xorshift32 generator behind the game's RANDOM opcode, and seeding it makes the engine fully deterministic. Same seed, same commands, bit-identical game. That single line turns a playthrough into a reproducible artifact.

Recording is a loop between an autonomous Fable agent and the real game running in the browser:

<pre class="mermaid">
sequenceDiagram
    participant F as Fable agent
    participant R as Recorder
    participant Z as Z-machine (seeded)
    F->>F: read the screen, write the thought
    F->>R: command + thought, one call
    R->>Z: submit through the real input path
    Z-->>R: turn output
    R->>R: store command, output hash, room
    Note over F,Z: the thought is final before the output exists
</pre>

The agent reads the game's output, writes its deliberation, and sends the thought and the command together. The recorder pushes the command through the same input path a human would use, waits for the turn to resolve, and stores the command with a hash of the output. The thought goes to a separate track, keyed by turn number.

That separation is strict: the recording is two files that are not allowed to touch. The run file is a pure engine artifact, commands and output hashes, nothing else. The thoughts live in the commentary file and render in their own overlay, never inside the game transcript. Tests enforce the boundary in both directions, plus one that insists every recorded turn carries a thought; the suite is at 90.

Replay boots a genuine game on the same seed and feeds it the recorded commands, so the scenes, map, and audio all react to real state. Every turn, the live output is hashed against the recording:

<pre class="mermaid">
flowchart LR
    run["run.json<br/>commands + hashes"] --> rp[Replayer]
    com["commentary.json<br/>thoughts by turn"] --> rp
    rp -->|stream thought, then type command| z["Z-machine<br/>same seed"]
    z --> g{"output hash<br/>matches?"}
    g -->|yes| rp
    g -->|no| halt[stop and say so]
    rp -.->|any keypress| you[you play from here]
</pre>

If a hash ever differs, playback stops and says so rather than pretend the run is still on the rails. It has never fired on a faithful replay. And because the replayed game is real, the take-over feature is nearly free: disable the feeder, enable the input, and the viewer is standing in a live dungeon holding whatever the model was carrying.

One last detail mattered more than I expected: pacing. The recorder keeps real timestamps, but they measure the agent's authoring time, minutes per move, unwatchable. So playback ignores them. Each thought streams word by word, the command only types after the thought finishes, movement turns breeze past, and a reading beat follows each output. Think first, then act, visibly, at 123 milliseconds per word.

The recording stops after 37 turns, at 54 of 350 points, two treasures carried and none banked. Fable offered to keep going, five or six more sittings to the full win. I said no. Its last thought on the tape closes the run better than I could: "Good place to pause. The empire waits."

<script type="module">
  import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
  mermaid.initialize({ startOnLoad: true, theme: "dark" });
</script>
