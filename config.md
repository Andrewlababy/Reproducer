---
default_mode:
notion_parent:
notion_database:
---

`default_mode` 只填 `report` 或 `assistant`。留空则每次先问。
`notion_parent` 填 Notion 父页面的 URL 或 ID。留空则建库前问一次，用户同意后写入。
`notion_database` 由技能在首次建库后写入（database URL 或 data source ID），之后都往这个库追加。
