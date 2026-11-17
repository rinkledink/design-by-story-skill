---
name: design-by-story
description: Builds a one-file clickable prototype (web, Android, iPhone, iPad) organised by user story, with ACs per story. Use to create, change, split or move stories, or sync them with a tracker.
---

# Design by Story

Version 1.2 (see CHANGELOG.md).

Build a single-file HTML prototype of a product, viewable on several devices, and
**broken down by user story** the same way it is broken down by device.

The top bar has two switchers:

- **Device**: e.g. Website · Android · iPhone. Same product, different frame.
- **Story**: "All stories" plus one entry per user story. Picking a story resets the
  shared data to that story's seed, sends every device to the story's start screen,
  and opens a panel with the story, priority, points, status, tracker link,
  acceptance criteria, walkthrough and notes.

Both selections live in the URL hash, so a link to one story on one device can be
pasted into a review or ticket. The shell supports two formats (see `HASH_STYLE`):
`#story=S2&device=iphone`, or `#story-S2--iphone` for hosts that only keep plain
`#name` hashes. It reads both either way.

## Workflow

1. **Gather inputs** (ask only for what's missing, in one message):
   - What the product is and who uses it.
   - **Existing designs**: screenshots, Figma frames or a design system. If they
     exist, the prototype matches them (see [Matching existing designs](#matching-existing-designs)).
   - Which devices to show. Default: Website + Android + iPhone. Tablet if field/kiosk use.
     Back-office and admin tools are usually website only.
   - The user stories, in any shape (a list, a backlog export, tracker items, feature docs).
     If they're in a tracker, read them from it (see [Working with a tracker](#working-with-a-tracker)).
   - **The order the stories will be built in.** Stories build on each other in that
     order (see [Stories build up in order](#stories-build-up-in-order)).
   - **AC format:** Given/When/Then, or plain rules? Default to Given/When/Then.
   - Priority (MoSCoW), story points and status, if they have them. Otherwise default
     to `should` / no points / `draft`.
   - The primary data model (main record + key fields), or infer it from the stories.

   Normalise every story to the [story format](#story-format). If the user has no
   stories, draft 3–6 from the product description and confirm them before building.
   If the user says you may rethink the stories, say what you changed and why.
2. **Check the stories** against the [quality checklist](#quality-checklist). Raise
   problems in one short list. Don't silently rewrite the user's wording.
3. **Plan screens per story.** For each story list the screens it touches on each
   device, **every region each screen shows** (cards, tables, footers, status labels),
   which story introduces each region, and the seed data the story needs. A story
   skips a device only when the product really works that way. Its `devices` list
   says so, and the switcher disables the other devices while it's selected.
4. **Copy `assets/shell.html`** to the user's chosen path (default
   `docs/reference/prototype.html`) and fill it in. Read
   `references/prototype-spec.md` for the visual and technical rules.
5. **Build screens, not stories.** Screens are shared; stories are entry points and
   seeds into them. Never duplicate a screen per story. If a story needs a variant,
   drive it from the seed data (see [Several data cases in one story](#several-data-cases-in-one-story)).
6. **Verify** in a browser (see [Verifying](#verifying)).
7. **Keep it in sync.** When a story is added or changed later, update its
   `#story-text` entry, its `BEHAVIOUR` entry, every device it lists and, if the user
   keeps a tracker, the tracker item, in the same round of changes.

## Story format

A story lives in two places in the shell, both keyed by its id:

- its **text** in the `#story-text` JSON block, shown read-only in the story panel;
- its **behaviour** in `BEHAVIOUR`, which is code only.

```html
<script type="application/json" id="story-text">
{
  "S1": {
    "title": "Save a search",
    "tracker": "1234",
    "points": 3,
    "story": "As a visitor, I can save a search, so that I hear about new matches without searching again.",
    "priority": "must",
    "status": "draft",
    "ac": [
      { "id": "S1-AC1", "text": "Given I’ve searched, when I tap Save search, then it appears under Saved" },
      { "id": "S1-AC2", "text": "Given I’ve searched, when I save it again, then Saved still shows it once" },
      { "id": "S1-AC4", "text": "Given nothing saved, then Saved shows “No saved searches yet.”" }
    ],
    "acNext": 4,
    "walkthrough": ["Search for “oak table”", "Tap Save search", "Open Saved"],
    "outOfScope": ["Email alerts (S4)"],
    "assumptions": ["Visitors must be signed in to save"],
    "questions": ["Is there a limit on saved searches?"],
    "changes": ["Removed S1-AC3: sorting saved searches moved to S5"]
  }
}
</script>
```

```js
const BEHAVIOUR = {
  S1: {
    devices: ['web', 'android', 'iphone'],         // which frames this story applies to
    start: { web: 'search', android: 'search', iphone: 'search' }, // screen per device
    adds: ['save-search'],                          // feature keys this story introduces
    seed: () => ({ user: SIGNED_IN, saved: [] }),  // fresh data for this story
  },
};
const BUILD_ORDER = ['S1', 'S2', 'S3'];            // the order stories are built in
```

Each story lists only what it **adds**. The shell works out its full `features` from
`BUILD_ORDER`: everything the story adds plus everything earlier stories added (see
[Stories build up in order](#stories-build-up-in-order)). Nobody maintains cumulative
lists by hand.

| Field | Rules |
|---|---|
| `story` | `As a <persona>, I can <action>, so that <benefit>`. The benefit is what the user gains, not a restatement of the action. Keep each part short: “I can set how long closing takes or how fast lots close”. The panel shows the three parts on separate lines. |
| `priority` | `must` · `should` · `could` · `won't` (MoSCoW). |
| `points` | Optional story points, as the team estimates them. Never invent points. |
| `tracker` | Optional tracker item id. Set `TRACKER_URL` in the shell to turn it into a link. |
| `status` | `draft` · `in review` · `agreed`. |
| `ac` | Objects `{ id, text }`. IDs are `<story>-AC<n>`, never renumbered, never reused. See below. |
| `acNext` | The highest AC number ever issued for this story. A new AC takes `acNext + 1`, and `acNext` is updated to match. |
| `walkthrough` | The demo script a reviewer clicks through, shown as “Try it”. **Not** acceptance criteria; keep it short. Concrete example values belong here. |
| `outOfScope`, `assumptions`, `questions` | Plain lists. Leave empty rather than inventing filler. Assumptions are only facts the story relies on; don't repeat what an AC already says. Record anything the user is unsure about as a question, and remove a question once it's answered (turn the answer into an AC or assumption). |
| `changes` | Bookkeeping notes: ACs removed, moved or split out (“Removed S2-AC4…”, “S4-AC4 moved to S6”, “Split out of S3”). Kept out of `assumptions` so those stay readable. |

`seed()` returns a fresh object on every call, so re-selecting a story always
restarts it cleanly. "All stories" uses `DEFAULT_SEED` and each device's home screen.

### Acceptance criteria

- **One testable behaviour per AC**, in the user's chosen format. Given/When/Then is
  one line: `Given <context>, when <action>, then <observable outcome>`.
- **"When" is only for an action** (a click, typing, a choice, scrolling). If the
  outcome is simply there to see, drop the when: `Given <context>, then <outcome>`,
  with the place in the then-clause ("then the footer shows…", not "when I look at
  the footer, then I see…").
- **Short.** Aim for one short line per clause, and cut words that don't change
  what's tested: "Given I’m editing", not "Given I’m editing the closing window".
- **Share the Given on purpose.** ACs with the same context use exactly the same
  Given text, so the panel shows them as one group (e.g. "Given the window hasn’t
  been changed" for three ACs). Order a story's ACs so each group sits together; IDs
  don't change, only the display order follows the groups.
- **Rules, not examples.** Write the rule ("the last lot closes exactly 3 hours after
  the first"), not one instance of it ("260 lots close over 3 h, 19:00–22:00").
  Put concrete values in the walkthrough. Use a worked example in an AC only when the
  user asks for one or the rule can't be stated clearly without it.
- **Exact copy:** when an AC is about a message, quote the message
  (`then I see “Nothing open right now.”`), never "a friendly message".
- **Outcomes, not styling:** "it shows as Done", not "a green pill appears". A visual
  detail is an AC only when it *is* the requirement.
- **Observable in this story.** If an outcome can only be seen through a later
  story's screen (e.g. recalculated times in a table that arrives in S3), record it
  as an assumption here and make it an AC in the later story.
- **Testable on one device.** Don't write ACs about other devices. Which devices a
  story covers belongs in `devices`.
- **IDs are permanent.** `S1-AC3` keeps its ID when edited. New ACs take
  `acNext + 1`. Deleted IDs are never handed out again, and nothing is renumbered.
  When an AC is removed, note it in `changes` ("Removed S2-AC4: changing the date
  is out of scope") so the gap is explained.

### Quality checklist

Check each story before building it, and again whenever it changes:

- Is the persona a real user role, and is the benefit something they'd say?
- Does it deliver value on its own, given the stories before it?
- Does every AC have an ID, a single behaviour, and an observable outcome **in this story**?
- Does the story text mention anything a later story introduces (saving, publishing,
  a control from another story)? If so, remove it or move it to `outOfScope` with the
  later story's id.
- Are empty, error, loading and limit cases covered, or listed under `questions`?
- Is it in the right place in `BUILD_ORDER`, after every story it relies on?
- Is every AC short, with a "when" only where there's an action, and do related ACs
  share their Given word for word?

## Stories build up in order

Stories are reviewed and built in order, S1 first. Each story shows **only what it
and the stories before it introduce**. This applies to everything on screen, not
just controls:

- **Regions are gated too.** A card, table, footer, side panel or status label that a
  later story introduces doesn't appear in an earlier one. Give each region a feature
  key (e.g. `closing-order` for the table, `save` for the save-state label) and put it
  in the `adds` of the story that introduces it. `BUILD_ORDER` turns it on for that
  story and every later one.
- **No forward references in copy.** An earlier story's screens and text don't
  mention a later capability ("Unsaved changes", "Not published", "Drag a lot" before
  dragging exists). Build hints and labels from `can(...)`. Copy that grows with a
  later feature switches on it too: Save shows "Schedule saved." and only adds
  "Buyers won’t see it until you publish." once the publish story's key is on.
- **Options belong to their story.** A choice a later or sibling story introduces
  (another option in a dropdown, a mode in a toggle) isn't offered or shown, not even
  as the current value. Seed each story so its starting state only uses what it and
  earlier stories introduce.
- **Interactions that need a later story move to that story.** If an AC needs
  something a later story adds (e.g. "replace your manual changes" needs manual
  reordering), move the AC to the later story (see [Moving ACs](#moving-acs-between-stories)).
- Data that every story genuinely needs to make sense (e.g. a lot's title) isn't gated.
  When in doubt, gate it.

## Story functionality is off until its story is selected

Every interaction a story introduces gets a feature key, listed in that story's
`adds`. `can('save-search')` is true while that story, or any later story in
`BUILD_ORDER`, is selected.

- **Render:** wrap the story's controls and regions in `can(...)` so they don't exist
  otherwise. Don't show them disabled. Reviewers see what the selected story adds, on
  top of what the stories before it built.
- **Handlers:** start each story handler with `if (!can('save-search')) return;` so
  nothing fires outside the story, even from a stale DOM or the console.
- **"All stories"** turns on every story's features at once: the whole product as it
  stands, on `DEFAULT_SEED`. Check that stories don't clash there.
- A story that depends on a later story's interaction belongs after it in `BUILD_ORDER`, or its AC moves to that story.

## Several data cases in one story

When one story must show the same screen with different data (a normal case, a large
case, an empty case), don't write three seeds or three stories. Seed a small list of
sample records and start the story on a list screen where the reviewer picks one
(e.g. auctions with 50 lots, 260 lots and none). Name the cases in the walkthrough.

## Matching existing designs

If the user provides designs, the product inside the frames matches them: layout,
spacing, typography, colours, copy, icons and control types. The visual spec's fonts
and colours apply only to the prototype's own controls (switchers and story panel)
and to products with no design yet. When a design shows a control the stories don't
cover (a Publish button, a Cards view), render it but make it inert with a toast such
as "Not part of this prototype". Use invented sample data even when the screenshot
shows real names.

## Changing stories

The prototype only shows stories; it never edits or saves them. All story changes
go through Claude, in the chat:

- **New story:** draft the story, ACs and questions, run the quality checklist with
  the user, then write the `#story-text` entry and the `BEHAVIOUR` entry and build
  the screens in one edit. A story never exists as text only.
- **Changed story** (wording, ACs, priority, points, status, notes): edit
  `#story-text`. If the change alters what the story does, update the screens and
  `BEHAVIOUR` to match in the same edit, and re-check every later story that builds on it.
- **Review feedback:** turn each comment into one of the changes above, or into an
  entry under `questions` if it's still open.
- **Answered questions:** remove the question, and add the answer as an AC or
  assumption.
- Always say what you built or changed, by story and AC ID.

### Moving ACs between stories

When an AC belongs in another existing story (not a split):

- If the behaviour is unchanged, it may keep its original ID in its new story.
- If it's reworded or merged with others, give it a new ID in the receiving story.
- Either way, add a note to `changes` in **both** stories, e.g. "S4-AC4 and S5-AC2 moved
  to S6 (now S6-AC5)", so tests and tickets can be traced.

## Splitting a story

When the user asks to split a story, or ACs could clearly ship separately, offer a
split once, with the pattern you'd use. A story with more than about 5 ACs is a hint,
not a rule. Don't split unless the user agrees.

1. **Choose a split pattern** and say which you used:
   - workflow steps (create → review → publish)
   - operations (add / edit / delete)
   - happy path first, then edge cases (empty, duplicate, limit reached)
   - by device or role (phone capture vs. website management)
   - by data variation (one item → many, one currency → several)
   - simple first, then extras (plain sort → saved sort preferences)

   Each child must still be a complete `As a… I can… so that…` with value of its own.
2. **Number children from the parent:** `S3` → `S3.1`, `S3.2`, … Remove the parent
   from `#story-text`, `BEHAVIOUR` and `BUILD_ORDER`, and put the children where it was. The
   shell's `readHash` sends an old link to `S3` to `S3.1`, so shared links keep
   working.
3. **Partition, don't duplicate:**
   - *Acceptance criteria.* Each parent AC moves to exactly one child **with its
     original ID**. If an AC straddles two children, rewrite it into two new ACs and
     note in `changes` which parent ID they replace.
   - *Feature keys.* Split a key if its interaction is split; each child `adds` only its part.
   - *Devices.* Each child lists only the devices it really touches.
   - *Seeds.* A later child's `seed()` starts from the state the earlier children
     would leave, so each child can be reviewed on its own.
   - *Notes.* Give each child the out-of-scope items, assumptions and questions that
     apply to it. Children inherit the parent's priority; points are re-estimated by
     the team, never split arithmetically.
4. **Update the screens:** re-gate any control that moved to a different key. Then
   check every child on every device it lists, and check that the parent's id
   redirects.
5. **Report the split** as a short table (child id, story, priority, devices, AC IDs).
   Mirror the split in the tracker if the user keeps one.

A story that's split *out* of another (a new sibling rather than children, e.g. "Scroll
to bottom" taken out of S3) takes the next free story number, and the moved ACs
follow [Moving ACs](#moving-acs-between-stories).

## Working with a tracker

If the user keeps stories in a tracker (Azure DevOps, Jira or similar), the tracker
is the master copy. Use the tracker's tools if they're connected.

- **Read before you build or write.** Fetch each item's title, description,
  acceptance criteria, points and child/parent links. Some trackers keep text in more
  than one field (e.g. Azure DevOps user stories can also have a "Repro Steps" field
  holding an older summary). Note every field that holds story text.
- **Writing back needs the user's go-ahead** for each round of changes. Before
  writing, say which items and fields you'll overwrite. Previous versions stay in the
  tracker's history, but say so.
- **Field mapping** (adjust to the tracker):
  - Title → the item title, prefixed with the prototype number (`S3 · Closing order table`)
    when the user wants them matched.
  - Story statement, out of scope, assumptions and open questions → description.
  - ACs → the acceptance criteria field, in the same grouped shape as the panel: a bold
    **Given** heading, then each AC's ID with **When** and **Then** on their own lines.
  - `changes` → a short "Changes" list at the end of the description.
  - Points → the story points field (Azure DevOps: `Microsoft.VSTS.Scheduling.StoryPoints`).
- **Link the prototype** at the top of each description with a deep link to that
  story. If the prototype is private, remind the user to share it.
- **References between stories** use tracker ids in the tracker and story ids in the
  prototype.
- **Report gaps**: stories in the prototype with no tracker item, tracker fields that
  still contradict the stories (template text, stale summaries), and items you were
  not asked to update but that now reference moved ACs.

## Verifying

Use a browser tool if you have one. Otherwise use Playwright (Python or Node) or
headless Chrome (`--dump-dom`, `--screenshot`, `--enable-logging=stderr`). Keep one
test script for the session that, for every story:

- opens it via its deep link on every device it lists and checks the start screen;
- clicks through the walkthrough and demonstrates every AC;
- checks **build-up**: no region, option, label or copy from a later story is present
  (search the rendered text for later stories' words, e.g. "Save", "Publish");
- checks switching device mid-story keeps state;
- collects console errors and page errors.

Re-run it after every change; regressions in other stories are common. Check
"All stories" has every control working together. Look at a screenshot of anything
matched to a design.

## Rules

- One HTML file, inline CSS/JS, no build step, no framework, no real backend. Open it
  straight from disk.
- Calls to a backend are simulated in memory. When an AC is about loading, paging,
  latency or a failed request, simulate it with a short timeout (300–500 ms) so the
  state can be seen.
- All data in one shared in-memory `state`; every device reads and writes it.
- Render functions take the device id and redraw from `state` (see shell).
- Use invented, neutral sample data. Never copy names, brands or content from other
  projects or from the user's screenshots into the prototype.
- Mobile frames render the same app markup; differences between Android and iPhone
  are chrome (status bar, notch/island, gesture bar), not separate apps, unless the
  user asks for platform-specific UI.
