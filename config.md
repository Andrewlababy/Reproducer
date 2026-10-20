---
default_mode:
destination:
destination_ref:
---

`default_mode`: `report` or `assistant`. Leave empty to ask the user at the start of each run.
`destination`: the tool that receives the records, for example `notion`, `jira`, `linear`, `slack`, `github-issues`, or `local` (keep local records only). Leave empty to ask the user once before the first record is written. Write the value after the user agrees.
`destination_ref`: the target location, for example a Notion database link, a Jira project key, or a Slack channel name. The skill writes this value after it finds or creates the target. All later records go to this location.
