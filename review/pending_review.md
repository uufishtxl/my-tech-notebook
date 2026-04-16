# 待复习错题本 (Pending Review)

## 2026-02-09
- [x] [Django] update() vs save() 信号 (update 不触发)
- [x] [Django] Migrations Non-null (需默认值)
- [ ] [DRF] SerializerMethodField (默认 `get_<field>`, 需熟悉 `method_name` 参数)
- [ ] [DRF] Serializer Source Assertion Error (同名字段不要加 source)
- [x] [API] Idempotency (PUT 幂等, POST 不幂等)
- [x] [Python] strip() 原理 (首尾字符集去除)
- [x] [Requests] raise_for_status (Fail Fast on 4xx/5xx)
- [x] [Dataclasses] frozen=True (Immutable -> Hashable)
- [x] [Regex] Greedy vs Non-Greedy (* vs *?)
- [x] [Data Types] List vs Tuple (Tuple 内存更优)
- [ ] [Vue] Setup vs Created (Setup 在 beforeCreate 前, 无 `this`)
- [x] [Vue] :key index 风险 (就地更新导致状态错乱)
- [x] [TS] Interface Merging (同名合并)
- [x] [CSS] Stacking Context (z-index 失效常见原因)
- [ ] [Vue] Computed in script setup (需 `.value`)
- [ ] [New] [Python] __slots__ 继承陷阱 (子类不继承 slots)
- [ ] [New] [DRF] Serializer 验证顺序 (Default->Field->Object->Model)

## 2026-02-08
- [x] [Database] B-Tree Index vs LIKE '%...' (前缀生效，包含失效)
- [x] [TS] Interface Merging (同名合并，Type 不支持)
- [x] [Vue] Composition API setup() vs created (setup 即 created)
- [x] [Regex] Greedy vs Non-Greedy (Review)
- [x] [Python] strip() 行为 (Review - 已纠正但需巩固)

## 2026-02-07
- [x] [Python] strip() 行为 (仅删除首尾字符，不删中间，注意参数)
- [x] [Regex] Greedy(*) vs Non-greedy(*?)
- [x] [Regex] match(只配开头) vs search(全文扫描)
- [x] [Requests] raise_for_status (Fail Fast on 4xx/5xx)
- [x] [Python] @dataclass frozen=True (Immutable & Hashable)
- [x] [CSS] 兄弟选择器 (选后用+, 选前用 has)
- [x] [Django] filter __in (匹配列表值)

## 2026-02-06
- [x] [Python] strip() 默认行为 (只删首尾空白，不删中间，传参非空白)
- [x] [Requests] raise_for_status() (必用！为了 Fail Fast)
- [x] [Vue] :key 原理 (不只是 index，要唯一 ID，否则 diff 错乱)
- [x] [Django] update() vs save() 信号触发 (update 不触发)
- [x] [Django] makemigrations Non-null (需默认值)
- [x] [API] multipart/form-data (由 boundary 分隔)
- [x] [Regex] Greedy(*) vs Non-greedy(*?)
- [x] [Regex] match(头) vs search(全)
- [x] [CSS] z-index 失效 (position static)

## 2026-02-05
- [x] [Django] makemigrations Non-null (提示 default value 或退出)
- [x] [Regex] Greedy vs Non-greedy (* vs *?)
- [x] [Django] update() signals (不触发 save/signals)
- [x] [API] Request Content-Type (multipart/form-data)
- [x] [Regex] match vs search (match 仅匹配开头)
- [x] [Python] strip() (仅去除首尾指定字符，注意空格)
- [x] [Requests] raise_for_status (4xx/5xx 抛出异常)
- [x] [CSS] z-index context (需要 non-static Position)
- [x] [Vue] key 原理 (VDOM diff 策略)

## 2026-02-03
...
