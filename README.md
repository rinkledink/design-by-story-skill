# design-by-story

A Claude Skill that builds a single-file, clickable HTML prototype of a product, viewable
on several devices (Website · Android · iPhone · iPad) and **broken down by user story**.

![A story selected on the website: the prototype on the left, the story with its acceptance criteria on the right](docs/screenshot-website.png)

Pick a device and a story from the top bar. The prototype then:

- resets to that story's data;
- opens each device on the story's start screen;
- shows the story alongside: priority, status, acceptance criteria (with IDs, in
  Given/When/Then), a walkthrough and notes.

A story's features only work while that story is selected; "All stories" turns them all on.
Links like `prototype.html#story=S2&device=iphone` open straight to a story on a device.

![The same prototype in the iPhone frame, with a different story selected](docs/screenshot-iphone.png)

The prototype shows stories but never edits them. To add, change or split a story, ask Claude.

## Install

In Claude Code:

```
/plugin install design-by-story --marketplace rinkledink/design-by-story-skill
```

Or copy the skill in by hand:

```sh
cp -r skills/design-by-story ~/.claude/skills/
```

Then ask Claude for a prototype, e.g. *"Prototype my app by story — here are the stories…"*.

## Contents

- `skills/design-by-story/SKILL.md`: workflow, story format, AC rules and how to split stories
- `skills/design-by-story/references/prototype-spec.md`: frames, tokens, type, components
- `skills/design-by-story/assets/shell.html`: working template with five example stories (open it to try)
- `.claude-plugin/`: plugin and marketplace manifests
