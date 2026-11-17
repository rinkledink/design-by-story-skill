# Changelog

## 1.2
- Acceptance criteria: "when" only for actions (Given…then otherwise), shorter wording, shared Given text so related ACs group.
- Story panel: As a / I can / So that rows, one card per Given with When and Then lines, folded notes with counts.
- New `changes` field for removed, moved and split ACs; assumptions keep only facts.
- Stories build up in order: each story lists what it `adds`; `BUILD_ORDER` works out the rest.
- Tracker: ACs written in the same grouped shape; description shorter than 200 characters.

## 1.1
- Stories build up in order: regions, options and copy from later stories stay hidden.
- Rules, not examples, in ACs; ACs only about what's observable in the story.
- Matching existing designs; several data cases in one story; moving ACs between stories.
- Working with a tracker (Azure DevOps, Jira): read first, field mapping, story points, prototype links.
- Template: `overflow: clip` frame, re-entrant render guard, fixed toasts, sticky dialogs, `points` and `tracker` fields, plain-hash links.
- Simulated loading, paging and errors when ACs need them.

## 1.0
- First version.
