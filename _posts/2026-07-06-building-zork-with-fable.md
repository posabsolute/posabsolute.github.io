---
layout: post
title: "Rebuilding Zork's world with Claude: Fable vs everything that came before"
---

<style>
     h1 {
        font-weight: normal;
        line-height: 1.5em;
        font-size: 28px;
        margin-bottom: 10px;
    }
    .post-title {
        margin-bottom: -0.5rem;
    }
    blockquote {
        margin-left: 10px;
        margin-right: 10px;
    }
    h2 { font-weight: normal; }
    .w {
        padding: 3em 1em;
    }
    .w img { border-radius: 4px; }
</style>

![](https://github.com/posabsolute/zork-ui/raw/main/docs/screenshots/hero-west-of-house.png)

Every model release ships with the same benchmark charts, and I have never known what those numbers mean for a real project. Last week I got a natural experiment instead: I built the same thing twice, once with Opus, once with Fable. So here it is, my position as of july '26, based on one project I can actually show you.

The project: the original 1980 Zork I running in the browser. Not a remake — the actual story file, byte for byte, on a real Z-machine. Around the untouched terminal, a shell: 110 animated pixel-art scenes that track the live game state, a generative soundtrack, an auto-drawn map. One rule from the start: observe, never interfere. The UI reads the Z-machine's memory and its output text. It never injects a command, never patches a rule. Strip the shell away and you have exactly the game your parents played, mid-session, unharmed.

You can [play it here](https://zork.arobase.co). Code is [on GitHub](https://github.com/posabsolute/zork-ui).

I built the first version with Opus and the second with Fable, on the same repo, with the same person giving directions, a few days apart. That is about as close to an apples-to-apples comparison as real work gets.

### The Opus day

The commit log from June 28 is the record. Project scaffold at 9:43. A vector-wireframe 3D viewport by noon. "Region-specific vector set-dressing (Another World style)" at 12:00. "Real 3D with hidden-surface removal" at 15:12. By evening it had pivoted again: "2D canvas room scenes; West of House as cyberpunk neon".

Every one of those passes was competent. The hidden-surface removal worked. The neon glow was a perfectly good neon glow. And every one of them looked like a kid's drawing. That is the phrase I used at the time, and I stand by it.

I didn't keep screenshots of the neon era. The commit log is the only witness. This turned out to be a mistake for writing this post, and it tells you something about how confident I was that any of it would survive.

Well, after a full day of it I had learned enough. I stopped iterating on the art direction. I had better things to do with my time.

To be fair to Opus: everything structural it built that day is still in the shipping product. The Z-machine integration, the room resolution, the command interception — that work was solid and none of it was thrown away. What it could not do was know what a 90s game screen looked like, or why. It executed every instruction and had no opinion of its own. Every increment of visual quality came out of my pocket: I squinted, I explained, it redrew. I was the taste and I was the QA, and that does not scale past about ten rooms.

### The Fable pass

A few days later I pointed Fable at the same repo and told it to redo everything.

The first thing it did was not draw. It wrote down the craft: a rules file naming Daggerfall and SCUMM as the reference era and Mark Ferrari's dithered landscapes as the standard — a 256-pixel offscreen buffer upscaled hard with smoothing off, ordered Bayer dithering, hue-shifted color ramps, selective outlining. Plus an anti-pattern list that read like a post-mortem of the Opus day: no glowing vector outlines, no flat silhouettes, no soft blur pretending to be glow.

Then it redid all 110 rooms, one at a time, and screenshot-verified every single one before calling it done — including both states of every interactive element, because the world in this thing reacts. Open the mailbox and the little door swings up. Kill the troll and the hall clears. Press the wrong button in the maintenance room and a pipe bursts, the water rising in the art turn by turn, exactly as fast as the game says it is:

![](https://github.com/posabsolute/zork-ui/raw/main/docs/screenshots/maintenance-flood.png)

The renderer reads the Z-machine's object tree directly, decoding object names out of raw memory, so the brown sack sits painted on the kitchen table until the moment you take it — or the moment the thief takes it from you.

![](https://github.com/posabsolute/zork-ui/raw/main/docs/screenshots/entrance-to-hades.png)

![](https://github.com/posabsolute/zork-ui/raw/main/docs/screenshots/cyclops-room.png)

The discipline is the part that changed how I work. When I asked, offhand, whether anything was worth refactoring, it split a 4,000-line scene file into region modules — and proved nothing changed by pixel-hashing all 110 scenes before and after, in two state configurations, byte-identical. When it rewrote 45 regex triggers into a declarative table, it ran the old code and the new table against a 57-case corpus of real game output and showed zero divergence before deleting anything. When production started returning 502s, it did not restart and pray; it root-caused the failure to an IPv4-only socket bind meeting Railway's IPv6 fabric after a sleep/wake cycle, fixed the bind, then deliberately let the app fall asleep and hit it again to prove the wake worked.

### Why Fable is better

Not speed. Not fewer syntax errors — neither model really makes those anymore. Three things, in order of importance:

**Taste.** Fable knew what good looked like before drawing, and could name where the standard came from. Knowing Mark Ferrari is not trivia. It is the difference between "make it retro" producing neon glow and "make it retro" producing ordered dither. You cannot review your way out of a model that does not know what it is aiming at.

**A verification reflex.** Verification is damn important. If the model proves its own work — a screenshot, a hash comparison, an equivalence corpus, a probe against production — you read evidence instead of hunting bugs, and you can hand over larger pieces. With Opus, checking was my job. With Fable, my job was reading the proof. This is the right way of doing things, like reviewing a diff that arrives with its tests already written and run.

**Coherence over a long project.** A week of "the mailbox blends into the terrain", "your map is a nightmare to understand", "the railing looks bad, rethink this completely" — and each note landed exactly once. It kept the whole system in its head: the art rules, the game data, the deploy quirks, the voice I wanted for the hint system.

To be clear about the limits: I was still the art director. I caught the mailbox, the map, the clues that gave too much away. It needed direction. But there is a world of difference between direction and supervision, and between these two models is where that line moved.

The wider point is not really about Opus. Think about what most people are actually running: the free tier of some chatbot, a fast cheap model in an autocomplete box, whatever their company licensed two years ago. When they say "AI can't do real work," that is the experience they are describing — and against that experience they are right. But the ladder above them is longer than they think. Opus sits well above the models most people have touched, it built the bones of this project in a single day, and it is _still_ not where things are. Everyone is debating a capability level that is two or three rungs below the current top of the ladder, and the rungs are not evenly spaced.

### Reception

I posted it on [r/ClaudeCode](https://www.reddit.com/r/ClaudeCode/comments/1umi07s/zork_ui_entirely_built_with_fable_5_with_110/) and [r/zork](https://www.reddit.com/r/zork/comments/1ulh4j8/imagining_zork_with_a_small_ui/), and more than 200 people have played so far.

The reaction was warmer than I expected, especially from the Zork crowd — the comment that mattered most called out that the visuals and audio _"mesh aesthetically with the text UI"_ without detracting from the text experience, which was the entire design thesis graded by the strictest possible audience. Players also made the product better within hours: a mobile layout shipped after someone couldn't `open mailbox` on his phone, and the room item list died after a commenter rightly said it _"takes all the fun out of realizing you didn't pick something up."_ My favorite comment predicted everyone will get addicted to Fable and then it will get shut off. I understand the fear.

Zork was written by four people at MIT on a PDP-10, inventing a genre with one MHz to work with. Forty-six years later I am art-directing a machine that redraws their world pixel by pixel and quotes their grue back at me.

I am going to be directing these models every day for many years to come. I want the one that knows what it is aiming at, and proves where it landed.

_It is pitch black. You are likely to be eaten by a grue._
