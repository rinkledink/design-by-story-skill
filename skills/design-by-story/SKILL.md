---
name: design-by-story
description: Use when the user wants a clickable product prototype or design mock-up (web app, phone app, tablet app, or several at once) and wants it organised by user story. Builds one self-contained HTML file with a device switcher (website / Android / iPhone / iPad) and a story switcher, so reviewers can pick a story and walk through exactly the screens and data that story needs on every device, alongside its acceptance criteria, priority, points, status and notes. Also use when adding, changing, moving or splitting stories in such a prototype, or syncing them with a tracker (Jira, Azure DevOps or similar).
---

# Design by Story

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
      { "id": "S1-AC1", "text": "Given I’ve searched, when I tap Save search, then the search appears under Saved" },
      { "id": "S1-AC2", "text": "Given a search is already saved, when I save it again, then Saved still shows it once" }
    ],
    "acNext": 2,
    "walkthrough": ["Search for “oak table”", "Tap Save search", "Open Saved"],
    "outOfScope": ["Email alerts (S4)"],
    "assumptions": ["Visitors must be signed in to save"],
    "questions": ["Is there a limit on saved searches?"]
  }
}
</script>
```

```js
const BEHAVIOUR = {
  S1: {
    devices: ['web', 'android', 'iphone'],         // which frames this story applies to
    start: { web: 'search', android: 'search', iphone: 'search' }, // screen per device
    features: ['save-search'],                      // this story's features plus every earlier story's
    seed: () => ({ user: SIGNED_IN, saved: [] }),  // fresh data for this story
  },
};
```

| Field | Rules |
|---|---|
| `story` | `As a <persona>, I can <action>, so that <benefit>`. The benefit is what the user gains, not a restatement of the action. |
| `priority` | `must` · `should` · `could` · `won't` (MoSCoW). |
| `points` | Optional story points, as the team estimates them. Never invent points. |
| `tracker` | Optional tracker item id. Set `TRACKER_URL` in the shell to turn it into a link. |
| `status` | `draft` · `in review` · `agreed`. |
| `ac` | Objects `{ id, text }`. IDs are `<story>-AC<n>`, never renumbered, never reused. See below. |
| `acNext` | The highest AC number ever issued for this story. A new AC takes `acNext + 1`, and `acNext` is updated to match. |
| `walkthrough` | The demo script a reviewer clicks through. **Not** acceptance criteria; keep it short. Concrete example values belong here. |
| `outOfScope`, `assumptions`, `questions` | Plain lists. Leave empty rather than inventing filler. Record anything the user is unsure about as a question, and remove a question once it's answered (turn the answer into an AC or assumption). |

`seed()` returns a fresh object on every call, so re-selecting a story always
restarts it cleanly. "All stories" uses `DEFAULT_SEED` and each device's home screen.

### Acceptance criteria

- **One testable behaviour per AC**, in the user's chosen format. Given/When/Then is
  one line: `Given <context>, when <action>, then <observable outcome>`.
- **Rules, not examples.** Write the rule ("a task due today shows as Due today"),
  not one instance of it ("Buy milk, due 14 March, shows as Due today").
  Put concrete values in the walkthrough. Use a worked example in an AC only when the
  user asks for one or the rule can't be stated clearly without it.
- **Exact copy:** when an AC is about a message, quote the message
  (`then I see “Nothing open right now.”`), never "a friendly message".
- **Outcomes, not styling:** "it shows as Done", not "a green pill appears". A visual
  detail is an AC only when it *is* the requirement.
- **Observable in this story.** If an outcome can only be seen through a later
  story's screen (e.g. a completed task leaving a filtered list that arrives in S3), record it
  as an assumption here and make it an AC in the later story.
- **Testable on one device.** Don't write ACs about other devices. Which devices a
  story covers belongs in `devices`.
- **IDs are permanent.** `S1-AC3` keeps its ID when edited. New ACs take
  `acNext + 1`. Deleted IDs are never handed out again, and nothing is renumbered.
  When an AC is removed, note it in `assumptions` ("S2-AC4 removed: undoing a
  completed task is out of scope") so the gap is explained.

### Quality checklist

Check each story before building it, and again whenever it changes:

- Is the persona a real user role, and is the benefit something they'd say?
- Does it deliver value on its own, given the stories before it?
- Does every AC have an ID, a single behaviour, and an observable outcome **in this story**?
- Does the story text mention anything a later story introduces (saving, publishing,
  a control from another story)? If so, remove it or move it to `outOfScope` with the
  later story's id.
- Are empty, error, loading and limit cases covered, or listed under `questions`?
- Are dependencies on earlier stories visible? (A shared feature key, plus a mention
  in `assumptions`, e.g. "Builds on S2".)

## Stories build up in order

Stories are reviewed and built in order, S1 first. Each story shows **only what it
and the stories before it introduce**. This applies to everything on screen, not
just controls:

- **Regions are gated too.** A card, table, footer, side panel or status label that a
  later story introduces doesn't appear in an earlier one. Give each region a feature
  key (e.g. `filter` for the filter bar, `assign` for the assignee column) and list
  it in every story from the one that introduces it onwards.
- **No forward references in copy.** An earlier story's screens and text don't
  mention a later capability ("Unassigned", "Show done", "Swipe to delete" before
  that story exists). Build hints and labels from `can(...)`.
- **Options belong to their story.** A choice a later or sibling story introduces
  (another option in a dropdown, a mode in a toggle) isn't offered or shown, not even
  as the current value. Seed each story so its starting state only uses what it and
  earlier stories introduce.
- **Interactions that need a later story move to that story.** If an AC needs
  something a later story adds (e.g. "the filter keeps a task I just
  assigned" needs assigning), move the AC to the later story (see [Moving ACs](#moving-acs-between-stories)).
- Data that every story genuinely needs to make sense (e.g. a task's title) isn't gated.
  When in doubt, gate it.

## Story functionality is off until its story is selected

Every interaction a story introduces gets a feature key, listed in that story's
`features`. `can('save-search')` is true only while a story listing it is selected.

- **Render:** wrap the story's controls and regions in `can(...)` so they don't exist
  otherwise. Don't show them disabled. Reviewers should see only what the selected
  story adds, on top of what earlier stories it builds on.
- **Handlers:** start each story handler with `if (!can('save-search')) return;` so
  nothing fires outside the story, even from a stale DOM or the console.
- **"All stories"** turns on every story's features at once: the whole product as it
  stands, on `DEFAULT_SEED`. Check that stories don't clash there.
- Because stories build up, each story lists its own feature keys plus every earlier
  story's (S3 in the shell lists `['add', 'complete', 'filter']`).

## Several data cases in one story

When one story must show the same screen with different data (a normal case, a large
case, an empty case), don't write three seeds or three stories. Seed a small list of
sample records and start the story on a list screen where the reviewer picks one
(e.g. projects with 5 tasks, 200 tasks and none). Name the cases in the walkthrough.

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
- Either way, add an assumption in **both** stories, e.g. "S2-AC3 moved to S3
  (now S3-AC4)", so tests and tickets can be traced.

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
   from both `#story-text` and `BEHAVIOUR` and put the children where it was. The
   shell's `readHash` sends an old link to `S3` to `S3.1`, so shared links keep
   working.
3. **Partition, don't duplicate:**
   - *Acceptance criteria.* Each parent AC moves to exactly one child **with its
     original ID**. If an AC straddles two children, rewrite it into two new ACs and
     note which parent ID they replace.
   - *Feature keys.* Split a key if its interaction is split.
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

A story that's split *out* of another (a new sibling rather than children, e.g. "Clear
completed tasks" taken out of S4) takes the next free story number, and the moved ACs
follow [Moving ACs](#moving-acs-between-stories).

## Working with a tracker

If the user keeps stories in a tracker (Jira, Azure DevOps or similar), the tracker
is the master copy. Use the tracker's tools if they're connected.

- **Read before you build or write.** Fetch each item's title, description,
  acceptance criteria, points and child/parent links. Some trackers keep text in more
  than one field (e.g. an older summary in a second text field). Note every field that holds story text.
- **Writing back needs the user's go-ahead** for each round of changes. Before
  writing, say which items and fields you'll overwrite. Previous versions stay in the
  tracker's history, but say so.
- **Field mapping** (adjust to the tracker):
  - Title → the item title, prefixed with the prototype number (`S3 · Filter by status`)
    when the user wants them matched.
  - Story statement, out of scope, assumptions and open questions → description.
  - ACs → the acceptance criteria field, as a list with their AC IDs and
    Given / when / then in bold.
  - Points → the tracker's story points field.
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
