---
name: design-by-story
description: Use when the user wants a clickable product prototype or design mock-up (web app, phone app, tablet app, or several at once) and wants it organised by user story. Builds one self-contained HTML file with a device switcher (website / Android / iPhone / iPad) and a story switcher, so reviewers can pick a story and walk through exactly the screens and data that story needs on every device, alongside its acceptance criteria, priority, status and notes. Also use when adding, changing or splitting stories in such a prototype.
---

# Design by Story

Build a single-file HTML prototype of a product, viewable on several devices, and
**broken down by user story** the same way it is broken down by device.

The top bar has two switchers:

- **Device**: e.g. Website · Android · iPhone. Same product, different frame.
- **Story**: "All stories" plus one entry per user story. Picking a story resets the
  shared data to that story's seed, sends every device to the story's start screen,
  and opens a panel with the story, priority, status, acceptance criteria,
  walkthrough and notes.

Both selections live in the URL hash (`#story=S2&device=iphone`), so a link to one
story on one device can be pasted into a review or ticket.

## Workflow

1. **Gather inputs** (ask only for what's missing, in one message):
   - What the product is and who uses it.
   - Which devices to show. Default: Website + Android + iPhone. Tablet if field/kiosk use.
   - The user stories, in any shape (a list, a backlog export, feature docs).
   - **AC format:** Given/When/Then, or plain rules? Default to Given/When/Then.
   - Priority (MoSCoW) and status, if they have them. Otherwise default to
     `should` / `draft`.
   - The primary data model (main record + key fields), or infer it from the stories.

   Normalise every story to the [story format](#story-format). If the user has no
   stories, draft 3–6 from the product description and confirm them before building.
2. **Check the stories** against the [quality checklist](#quality-checklist). Raise
   problems in one short list. Don't silently rewrite the user's wording.
3. **Plan screens per story.** For each story list the screens it touches on each
   device and the seed data it needs. A story skips a device only when the product
   really works that way (e.g. admin only on the website). Its `devices` list says
   so, and the switcher disables the other devices while it's selected.
4. **Copy `assets/shell.html`** to the user's chosen path (default
   `docs/reference/prototype.html`) and fill it in. Read
   `references/prototype-spec.md` for the visual rules (frames, tokens, type, cards,
   pills, toasts, drawers, icons, copy tone).
5. **Build screens, not stories.** Screens are shared; stories are entry points and
   seeds into them. Never duplicate a screen per story. If a story needs a variant,
   drive it from the seed data.
6. **Verify** in a browser. Use a browser tool if you have one. Otherwise use
   headless Chrome: `--dump-dom "file://…/prototype.html#story=S2&device=iphone"` to
   check what rendered, `--screenshot` to look at it, and `--enable-logging=stderr`
   to catch console errors. Check that:
   - every story on every device it lists opens on the right screen;
   - every AC can be demonstrated, and the walkthrough can be clicked through;
   - switching device mid-story keeps state;
   - no story's controls appear under another story, and "All stories" has every
     story's controls working together;
   - the console is clean.
7. **Keep it in sync.** When a story is added or changed later, update its
   `#story-text` entry, its `BEHAVIOUR` entry and every device it lists in the same edit.

## Story format

A story lives in two places in the shell, both keyed by its id:

- its **text** in the `#story-text` JSON block, shown read-only in the story panel;
- its **behaviour** in `BEHAVIOUR`, which is code only.

```html
<script type="application/json" id="story-text">
{
  "S1": {
    "title": "Save a search",
    "story": "As a visitor, I can save a search, so that I hear about new matches without searching again.",
    "priority": "must",
    "status": "draft",
    "ac": [
      { "id": "S1-AC1", "text": "Given I searched for \"oak table\", when I tap Save search, then it appears under Saved" },
      { "id": "S1-AC2", "text": "Given that search is already saved, when I save it again, then Saved still shows it once" }
    ],
    "acNext": 2,
    "walkthrough": ["Search for \"oak table\"", "Tap Save search", "Open Saved"],
    "outOfScope": ["Email alerts"],
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
    features: ['save-search'],                      // what this story switches on
    seed: () => ({ user: SIGNED_IN, saved: [] }),  // fresh data for this story
  },
};
```

| Field | Rules |
|---|---|
| `story` | `As a <persona>, I can <action>, so that <benefit>`. The benefit is what the user gains, not a restatement of the action. |
| `priority` | `must` · `should` · `could` · `won't` (MoSCoW). |
| `status` | `draft` · `in review` · `agreed`. |
| `ac` | Objects `{ id, text }`. IDs are `<story>-AC<n>`, never renumbered, never reused. See below. |
| `acNext` | The highest AC number ever issued for this story. A new AC takes `acNext + 1`, and `acNext` is updated to match. Update it whenever you add an AC. |
| `walkthrough` | The demo script a reviewer clicks through. **Not** acceptance criteria; keep it short. |
| `outOfScope`, `assumptions`, `questions` | Plain lists. Leave empty rather than inventing filler. Record anything the user is unsure about as a question. |

`seed()` returns a fresh object on every call, so re-selecting a story always
restarts it cleanly. "All stories" uses `DEFAULT_SEED` and each device's home screen.

### Acceptance criteria

- **One testable behaviour per AC**, in the user's chosen format. Given/When/Then is
  one line: `Given <context>, when <action>, then <observable outcome>`.
- **Exact copy:** when an AC is about a message, quote the message
  (`then I see "Nothing open right now."`), never "a friendly message".
- **Outcomes, not styling:** "it shows as Done", not "a green pill appears". A visual
  detail is an AC only when it *is* the requirement.
- **Testable on one device.** Don't write ACs about other devices ("appears on every
  device", "web-only", "phones can't change it"). Which devices a story covers
  belongs in `devices`, and syncing between devices is the backend's concern.
- **IDs are permanent.** `S1-AC3` keeps its ID when edited, moved or split into a
  child story, so it can be traced to tests and tickets. New ACs take `acNext + 1`.
  Deleted IDs are never handed out again, and nothing is renumbered.

### Quality checklist

Check each story before building it, and again whenever it changes:

- Is the persona a real user role, and is the benefit something they'd say?
- Does it deliver value on its own (independent), and is it small enough to demo in
  under a minute? If not, [split it](#splitting-a-story).
- Does every AC have an ID, a single behaviour, and an observable outcome?
- Are empty, error and limit cases covered, or listed under `questions`?
- Are dependencies on other stories visible? (A shared feature key, plus a mention
  in `assumptions`, e.g. "Builds on S2".)

## Story functionality is off until its story is selected

Every interaction a story introduces gets a feature key, listed in that story's
`features`. `can('save-search')` is true only while a story listing it is selected.

- **Render:** wrap the story's controls in `can(...)` so they don't exist otherwise
  (no button, no form, no menu item). Don't show them disabled. Reviewers should see
  only what the selected story adds.
- **Handlers:** start each story handler with `if (!can('save-search')) return;` so
  nothing fires outside the story, even from a stale DOM or the console.
- **"All stories"** turns on every story's features at once: the whole
  product as it stands, on `DEFAULT_SEED`. Check that stories don't clash there
  (e.g. two controls in the same spot).
- A story that depends on another story's interaction lists both keys
  (e.g. "Filter by status" lists `['filter', 'complete']` so ticking a task off
  can be shown leaving the filtered list).
- Data a story only *displays* (e.g. an assignee's initials) is not gated; only
  the ability to change it is.

## Changing stories

The prototype only shows stories; it never edits or saves them. All story changes
go through Claude, in the chat:

- **New story:** draft the story, ACs and questions, run the quality checklist with
  the user, then write the `#story-text` entry and the `BEHAVIOUR` entry and build
  the screens in one edit. A story never exists as text only. If it did, the shell
  would leave it out of the switcher and log an error in the console.
- **Changed story** (wording, ACs, priority, status, notes): edit `#story-text`. If
  the change alters what the story does (a new AC, a different walkthrough), update
  the screens and `BEHAVIOUR` to match in the same edit.
- **Review feedback:** when the user passes on reviewers' comments, turn each into
  one of the changes above, or into an entry under `questions` if it's still open.
- Always say what you built or changed, by story and AC ID.

If the user keeps stories in a tracker (Jira or similar), the tracker is the master
copy. Mirror its wording here rather than editing stories only in the prototype.

## Splitting a story

When the user asks to split a story, or a story has more than about 5 ACs, more than
one main feature key, or ACs that could ship separately, split it.

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
   shell's `readHash` sends an old `#story=S3` link to `S3.1`, so shared links keep
   working.
3. **Partition, don't duplicate:**
   - *Acceptance criteria.* Each parent AC moves to exactly one child **with its
     original ID** (`S3-AC2` stays `S3-AC2` inside `S3.1`). If an AC straddles two
     children, rewrite it into two new ACs (`S3.1-AC1`, `S3.2-AC1`) and note which
     parent ID they replace.
   - *Feature keys.* Split a key if its interaction is split
     (`filter` → `filter-status`, `filter-counts`). A child may also list a sibling's
     key when it builds on it, as in the dependency rule above.
   - *Devices.* Each child lists only the devices it really touches.
   - *Seeds.* A later child's `seed()` starts from the state the earlier children
     would leave (e.g. S3.2 seeds some items already saved), so each child can be
     reviewed on its own.
   - *Notes.* Give each child the out-of-scope items, assumptions and questions that
     apply to it. Children inherit the parent's priority unless the user says otherwise.
4. **Update the screens:** re-gate any control that moved to a different key. Then
   check every child on every device it lists, and check that the parent's id
   redirects.
5. **Report the split** as a short table (child id, story, priority, devices, AC IDs)
   so the user can check that every parent AC landed somewhere. Mirror the split in
   their backlog or feature docs if they keep stories there.

## Rules

- One HTML file, inline CSS/JS, no build step, no framework, no backend. Open it
  straight from disk.
- All data in one shared in-memory `state`; every device reads and writes it.
- Render functions take the device id and redraw from `state` (see shell).
- Use invented, neutral sample data. Never copy names, brands or content from other
  projects into the prototype.
- Mobile frames render the same app markup; differences between Android and iPhone
  are chrome (status bar, notch/island, gesture bar), not separate apps, unless the
  user asks for platform-specific UI.
