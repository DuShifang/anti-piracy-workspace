# 历史规则设计｜v0.9 参考入口

只有当前需求确实值得评估规则驱动适配时才读后文相关章节。可以完全不用它，也可以只借鉴局部思想。默认选型先看 [候选方案](CANDIDATE_SOLUTIONS.md)；核心行为与验证要求以主准则和 [验收准则](ACCEPTANCE_CRITERIA.md)为准。

以下逐字节保留 v0.8.2 的旧文件。它是历史设计混合后续局部修订的快照，不是当前可直接加载、已实现全部字段的 Schema，也不是“每一段示例都已经 PoC 验证”的证明。本轮未运行规则引擎、未升级任何执行器。

## 必须先理解的兼容边界

后文“必须读取本文件”“不得直接写站点专属代码”“第一批规则”“所有实验都通过”等属于当时所选协议的局部开发安排，不要求当前项目复制。采用具体子集前说明当前执行器支持什么，不识别的关键字段不能默默忽略。

旧示例仍标 `schemaVersion: 1`，提交示例已写有 `mode: user-approved / finalAction: adapter`。这只证明文档中有这些字段，不能证明任何旧执行器支持；需要采用时验证兼容及迁移，不靠 MD 更改假装软件已经升级。未经本次授权的自动提交仍禁止，授权后的机械自动化按主准则处理。通用项目的普通维修也不受“只能改 candidate.yaml”这类历史限制。

## v0.9.1 共用填表约束

后文投诉字段、uploads、submit 及通用执行器示例仅供历史研究，不能用来为个人作者重新建设平行填表/提交流程。当前填表使用 [共用成品 0.3.1](SHARED_EXTENSION.md)，协议中不接受 selector、脚本、allowedHosts 或 approved；真实最终提交禁用。搜索等非填表能力仍可按需借鉴此历史参考。本段不改写旧原文，也不宣称旧 Schema 与共用协议兼容。

<!-- V082_ORIGINAL_BEGIN: references/RULES_SPEC.md -->

# RULES_SPEC.md｜可选规则系统参考实现

> Status: optional reference  
> Origin: legacy `RULES_SPEC.md v0.1`  
> Role: 复杂网页适配与 AI 可维护规则系统的一种参考设计，不是当前项目的默认架构  
> Primary guidance: 当前工作区的 `AGENTS.md` 与 `BUILD_YOUR_OWN_ANTIPIRACY_APP_PROMPT.md`

---

# 阅读前提：不要因为本文件存在就实现它

本文件保留的是此前为 Local-First 网文反盗版助手研究、设计并用于 PoC 验证的一套**规则驱动网页适配方案**。它的价值是：当当前作者的软件真的遇到多网站、频繁改版、复杂页面交互、规则可测试、AI 后续维修等问题时，Coding Agent 可以直接借鉴已经研究过的抽象和安全边界，而不必重新从零踩坑。

**本文件不是当前生成软件必须遵循的产品协议，也不是统一软件蓝图。**

在阅读后续技术内容前，先遵守以下解释规则：

1. **先由用户需求决定是否需要规则系统。** 不要因为工作区里有 `RULES_SPEC.md`，就默认创建 Rule Engine、YAML DSL、Step DSL、统一 Schema、Fixture 目录或规则维修链。
2. **简单方案优先。** 如果当前软件只适配少量稳定来源，普通模块、适配器、脚本或直接代码已经足够可靠，就采用更简单的实现。
3. **允许只借鉴局部。** 可以只采用 selector fallback、Detection、URL 规范化、fixture 测试、candidate 验证等某些思想，不需要整套协议一起实现。
4. **后文的“必须 / 应 / 禁止”具有局部作用域。** 只有 Coding Agent 已经根据当前项目需求选择采用本参考方案或其中某个子系统时，这些约束才适用于那个被采用的子系统；它们不能反过来证明当前项目必须采用该子系统。
5. **安全原则可以独立复用。** 不执行远程任意脚本、不绕验证码或访问控制、敏感数据最小化、候选修改独立验证、正式修改可回滚等原则，即使不采用规则系统，也值得在其他实现中保留。
6. **历史方案不是未来包袱。** 当前 Coding Agent 可以采用更简单或更合适的新设计，只要满足主开发准则中的隐私、安全、可测试、可维护和人工确认要求。

## 什么时候值得参考这份文件

比较适合参考：

- 当前软件需要适配多个结构不同的网站；
- selector、翻页、二次链接解析等逻辑预计会频繁变化；
- 希望网站改版时优先修改小范围适配配置，而不是核心程序；
- 希望 Coding AI 能针对网站规则做隔离维修和独立测试；
- 需要 fixture、Schema 或其他可重复验证机制来降低维修风险。

通常没有必要引入整套方案：

- 只有一两个稳定来源；
- 网站提供更简单、稳定且允许使用的接口；
- 当前需求只是轻量搜索或人工辅助；
- 用户最看重极简、便携，而规则框架带来的维护成本反而更高；
- 当前功能根本不涉及复杂网页自动化。

## 关于“经过验证”的含义

此前的相关 PoC 已经实际验证过其中若干关键思路，例如规则驱动搜索、结果作用域、二次链接解析、分页与候选规则经 AI 修改后由应用独立复验等。  
但这不表示下面每一个字段、每一种规则类型、每一段示例都已经在所有真实网站上逐项验证，也不表示未来项目必须保持这套 Schema 不变。

因此，应把本文理解为：

> **一套有实测经验支撑的工程参考，而不是需要机械实现的标准。**

---

# 以下为原 RULES_SPEC v0.1 技术内容

从这里开始保留原有技术设计。文中的“本协议”均应按照上面的阅读前提理解。

---

# 0. 设计目标

本协议用于描述反盗版助手中所有容易因网站改版而失效的行为。

目标不是把网页自动化写成“另一门完整编程语言”，而是提供一套足够表达常见网页工作流、同时保持安全、可审计、可测试的声明式规则。

规则应尽可能回答：

```text
去哪里
↓
何时执行
↓
找什么
↓
如何提取
↓
如何翻页
↓
何时停止
↓
出现异常怎么办
↓
结果如何映射到标准数据结构
```

本协议优先服务四个目标：

1. **可读**：普通开发者和 Coding Agent 能直接阅读 YAML。
2. **可修**：网站变化时优先修改规则，而不是核心代码。
3. **可测**：每条规则可以通过 fixture 和 live smoke test 验证。
4. **安全**：禁止规则远程执行任意代码，禁止把自动化升级成验证码绕过或访问控制规避。

---

# 1. 规则分类

v0.1 定义四种顶层规则：

```text
search
site
complaint
panel
```

目录建议：

```text
rules/
├─ search/
├─ sites/
├─ complaint/
└─ panels/
```

另外允许：

```text
rules/prompts/
```

用于 AI Prompt 模板，但 Prompt 不属于本协议的网页执行规则。

---

# 2. 通用顶层结构

所有规则必须包含：

```yaml
schemaVersion: 1
type: search
id: bing
name: Bing
enabled: true
```

完整通用头：

```yaml
schemaVersion: 1

type: search | site | complaint | panel

id: unique-id
name: Human Readable Name

enabled: true

description: optional text

version: 1

maintainer:
  mode: local
  note: optional

match:
  hosts: []
  urlPatterns: []

capabilities: []

metadata:
  createdAt: optional
  updatedAt: optional
  notes: optional
```

## 2.1 `schemaVersion`

协议版本。

v0.1 固定：

```yaml
schemaVersion: 1
```

## 2.2 `type`

允许：

```text
search
site
complaint
panel
```

## 2.3 `id`

要求：

- 全局稳定；
- 小写；
- 推荐 `kebab-case`；
- 不随显示名称变化。

例如：

```text
bing
baidu
iwangpan
example-novel-site
example-copyright-platform
```

## 2.4 `version`

单条规则自身版本。

每次正式更新规则后递增。

## 2.5 `enabled`

仅表示默认启用状态。

用户本地设置可以覆盖。

---

# 3. 模板变量

规则允许使用有限模板变量。

统一语法：

```text
{{variable}}
```

禁止同时兼容多套 `{keyword}` / `{{keyword}}` 语法。

v0.1 统一只使用双花括号。

标准变量：

```text
{{keyword}}
{{page}}
{{offset}}
{{first}}
{{work.title}}
{{work.author}}
{{currentUrl}}
```

URL 模板示例：

```yaml
url: "https://example.com/search?q={{keyword}}&page={{page}}"
```

所有变量必须来自运行时白名单。

禁止规则读取任意环境变量。

---

# 4. SelectorSpec

网页元素定位使用 `SelectorSpec`。

最简单形式：

```yaml
selector: "h2 a"
```

推荐形式：

```yaml
selectors:
  - "h2 a"
  - ".result-title a"
  - "a[data-title]"
```

语义：

> 按顺序尝试，使用第一个成功匹配的 selector。

完整形式：

```yaml
selector:
  type: css
  value: "h2 a"
```

或：

```yaml
selector:
  type: xpath
  value: "//h2/a"
```

v0.1 支持：

```text
css
xpath
```

默认：

```text
css
```

## 4.1 多 selector fallback

```yaml
selectors:
  - type: css
    value: ".result a"

  - type: xpath
    value: "//div[@class='result']//a"
```

Agent 应优先使用稳定 selector。

不推荐：

```text
div:nth-child(7) > div:nth-child(1) ...
```

除非页面确实不存在更稳定的属性。

---

# 5. ExtractorSpec

Extractor 将 DOM 元素转换成标准值。

基础结构：

```yaml
extract:
  source: text
```

支持：

```text
text
html
href
prop
attr
urlParam
base64UrlParam
jsonAttr
regex
citeUrl
```

---

## 5.1 text

```yaml
extract:
  source: text
```

返回：

```text
textContent
```

默认：

```yaml
trim: true
```

---

## 5.2 html

```yaml
extract:
  source: html
```

返回 `innerHTML`。

应谨慎使用。

---

## 5.3 href

```yaml
extract:
  source: href
```

等价于优先读取绝对 URL。

---

## 5.4 prop

```yaml
extract:
  source: prop
  name: href
```

---

## 5.5 attr

```yaml
extract:
  source: attr
  name: data-url
```

---

## 5.6 urlParam

用于从 URL 查询参数中恢复真实链接：

```yaml
extract:
  source: urlParam
  input:
    source: href
  param: target
  decode: true
```

---

## 5.7 base64UrlParam

```yaml
extract:
  source: base64UrlParam
  input:
    source: href
  param: url
```

执行器负责：

1. 读取参数；
2. Base64 decode；
3. 验证结果是否为合法 URL。

---

## 5.8 jsonAttr

用于属性中存放 JSON 的情况：

```yaml
extract:
  source: jsonAttr
  attr: data-info
  path: target.url
```

不得允许任意 JS 表达式。

---

## 5.9 regex

```yaml
extract:
  source: text
  regex:
    pattern: "https?://[^\\s]+"
    group: 0
```

---

## 5.10 citeUrl

用于搜索引擎显示文本中出现的 URL。

```yaml
extract:
  source: citeUrl
```

执行器应只进行受限 URL 恢复。

---

# 6. TransformSpec

Extractor 后允许有限转换。

```yaml
transform:
  - type: trim
  - type: decodeURIComponent
  - type: removeTrackingParams
```

v0.1 允许：

```text
trim
lowercase
uppercase
decodeURIComponent
removePrefix
removeSuffix
regexReplace
removeTrackingParams
normalizeUrl
```

示例：

```yaml
transform:
  - type: trim

  - type: regexReplace
    pattern: "^跳转："
    replacement: ""
```

禁止：

```text
eval
javascript
shell
```

---

# 7. FieldSpec

用于描述标准字段。

例如搜索结果：

```yaml
fields:
  title:
    selectors:
      - "h2 a"
    extract:
      source: text

  url:
    selectors:
      - "h2 a"
    extract:
      source: href

  snippet:
    selectors:
      - ".caption"
      - ".snippet"
    extract:
      source: text
    optional: true
```

通用字段：

```yaml
optional: false
maxLen: 500
default: null
```

---

# 8. Step DSL

Step DSL 是 v0.1 的核心。

用于描述有限网页交互。

所有 Step 必须来自白名单。

允许：

```text
click
type
wait
waitForSelector
navigate
extract
newPage
goBack
scroll
screenshot
saveUrl
saveHtml
```

禁止：

```text
javascript
eval
shell
exec
downloadAndRun
```

---

# 9. Step: click

```yaml
- type: click
  selector: ".search-button"
```

可选：

```yaml
timeoutMs: 5000
optional: false
```

---

# 10. Step: type

```yaml
- type: type
  selector: "#keyword"
  value: "{{keyword}}"
  clear: true
```

---

# 11. Step: wait

```yaml
- type: wait
  ms: 1000
```

必须有上限。

建议全局限制：

```text
单次 wait <= 30000ms
```

---

# 12. Step: waitForSelector

```yaml
- type: waitForSelector
  selector: ".result-item"
  timeoutMs: 10000
```

---

# 13. Step: navigate

```yaml
- type: navigate
  url: "https://example.com/search?q={{keyword}}"
```

只能导航到规则允许的 host 或明确声明的外部目标。

---

# 14. Step: extract

```yaml
- type: extract
  selector: "a[data-url]"
  extract:
    source: attr
    name: data-url
  saveAs: resolvedUrl
```

---

# 15. Step: newPage

用于点击后打开新页面：

```yaml
- type: newPage
  click:
    selector: ".open-result"

  waitForLoad: true

  extract:
    selector: ".download-link"
    extract:
      source: href

  saveAs: resolvedUrl

  closeAfter: true
```

如果点击没有打开新页面，但当前页发生导航，执行器可以按实现允许兼容处理，但必须有明确超时和恢复逻辑。

---

# 16. Step: goBack

```yaml
- type: goBack
  waitForLoad: true
```

---

# 17. Step: scroll

```yaml
- type: scroll
  direction: down
  amount: viewport
```

允许：

```text
viewport
page
pixels
```

若使用 pixels：

```yaml
pixels: 800
```

---

# 18. Step: screenshot

```yaml
- type: screenshot
  saveAs: "debug-after-search"
```

规则不得指定任意绝对磁盘路径。

---

# 19. Step: saveUrl / saveHtml

```yaml
- type: saveUrl
  saveAs: currentUrl
```

```yaml
- type: saveHtml
  saveAs: pageHtml
```

这些值只能写入运行时上下文或诊断系统。

---

# 20. Step 条件

v0.1 支持简单条件：

```yaml
when:
  selectorExists: ".consent-button"
```

例如：

```yaml
- type: click
  selector: ".consent-button"
  optional: true
  when:
    selectorExists: ".consent-button"
```

v0.1 不支持任意布尔表达式语言。

---

# 21. DetectionSpec

检测页面状态。

示例：

```yaml
detect:
  - id: captcha
    severity: pause_for_user

    any:
      - titleContains: "验证"
      - urlContains: "/captcha"
      - selectorExists: ".captcha"
      - textContains: "请完成验证"

    message: "搜索引擎要求人工验证。"
```

支持：

```text
titleContains
titleMatches
urlContains
urlMatches
textContains
htmlContains
selectorExists
selectorMissing
```

severity：

```text
info
retryable
terminal
pause_for_user
```

---

# 22. Detection 行为

## info

记录但继续。

## retryable

允许按 RetryPolicy 重试。

## terminal

当前任务终止。

## pause_for_user

暂停任务并等待用户。

验证码、登录、安全验证必须使用：

```text
pause_for_user
```

不得自动绕过。

---

# 23. RetryPolicy

```yaml
retry:
  maxAttempts: 2
  delayMs: 3000
```

禁止无限重试。

---

# 24. DelaySpec

```yaml
delays:
  page:
    minMs: 1200
    maxMs: 2500

  turnPage:
    minMs: 1500
    maxMs: 3000

  engine:
    minMs: 2000
    maxMs: 5000
```

延迟用于：

- 页面稳定；
- 控制资源占用；
- 避免过快连续操作。

不得把 DelaySpec 宣传为绕过风控能力。

---

# 25. PaginationSpec

分页统一结构：

```yaml
pagination:
  type: url | click | scroll
```

---

# 26. URL Pagination

```yaml
pagination:
  type: url

  template: "https://example.com/search?q={{keyword}}&page={{page}}"

  startPage: 1
  step: 1
  maxPages: 5
```

---

# 27. Click Pagination

```yaml
pagination:
  type: click

  next:
    selectors:
      - ".next"
      - "a[rel='next']"

  waitFor:
    selector: ".result-item"

  maxPages: 5
```

终止条件：

```yaml
end:
  selectorMissing: ".next"
```

或：

```yaml
end:
  selectorHasClass:
    selector: ".next"
    class: disabled
```

---

# 28. Scroll Pagination

```yaml
pagination:
  type: scroll

  scrollWaitMs: 1200

  end:
    selectorExists: ".no-more"

  maxTurns: 20
```

必须有：

```text
maxTurns
```

或可靠 endSignal。

推荐两者都存在。

---

# 29. HookSpec

Hook 是受限白名单动作。

示例：

```yaml
hooks:
  - phase: after_extract
    action: screenshot

  - phase: on_error
    action: emitDiagnostic
```

允许 phase：

```text
before_search
after_search
before_extract
after_extract
before_turn_page
after_turn_page
on_error
on_blocked
```

允许 action 由应用注册。

v0.1 推荐内置：

```text
screenshot
saveUrl
saveHtml
emitDiagnostic
```

规则不得携带 action 实现代码。

---

# 30. Search Rule

`type: search`

目标：

> 根据关键词从搜索引擎、网盘搜索、内容搜索站等来源获得候选结果。

基础结构：

```yaml
schemaVersion: 1
type: search
id: example-search
name: Example Search
enabled: true

entry:
  mode: url
  baseUrl: "https://example.com"

search:
  url: "https://example.com/search?q={{keyword}}"

results:
  passes: []

pagination: {}

detect: []

limits: {}

delays: {}
```

---

# 31. Search Entry

支持：

```text
url
interaction
```

## url

```yaml
entry:
  mode: url
```

直接使用 search URL。

## interaction

用于必须先打开首页、输入框、点击的站点：

```yaml
entry:
  mode: interaction
  baseUrl: "https://example.com"

  steps:
    - type: click
      selector: ".consent"
      optional: true

    - type: type
      selector: "#keyword"
      value: "{{keyword}}"

    - type: click
      selector: "#search"
```

这统一替代历史上的 `replay` 等命名。

---

# 32. Search Result Pass

允许多组解析策略：

```yaml
results:
  passes:
    - id: desktop-main

      containers:
        - ".result"
        - "li.search-result"

      fields:
        title:
          selectors:
            - "h2 a"
          extract:
            source: text

        url:
          selectors:
            - "h2 a"
          extract:
            source: href

        snippet:
          selectors:
            - ".snippet"
          extract:
            source: text
          optional: true
```

第二组 fallback：

```yaml
    - id: alternate-layout
      containers:
        - "article"

      fields:
        ...
```

执行器可以：

- 合并多个 pass；
- 统一去重。

---

# 33. Search Link Resolution

有些搜索结果 URL 不是最终 URL。

字段可以定义：

```yaml
url:
  selectors:
    - ".result-button"

  extract:
    source: text

  resolve:
    steps:
      - type: click
        selector: ".result-button"

      - type: extract
        selector: "a[data-url]"
        extract:
          source: attr
          name: data-url
        saveAs: resolvedUrl
```

推荐最终标准字段始终为：

```text
url
```

---

# 34. Search Result Post Processing

允许：

```yaml
postProcess:
  dropFirst: 0
  dropLast: 0
```

避免布尔：

```text
dropFirst: true
```

推荐明确数量。

---

# 35. Search Domain Filtering

```yaml
filters:
  excludeDomains:
    - example-official.com

  includeDomains: []
```

应用层还会叠加：

- officialDomains
- userWhitelist
- userBlockedDomains

规则自己的 excludeDomains 不是唯一来源。

---

# 36. Search Limits

```yaml
limits:
  maxPages: 5
  maxResults: 200
  timeoutMs: 120000
```

必须限制搜索规模。

---

# 37. Search Rule 完整示例

```yaml
schemaVersion: 1
type: search

id: example-search
name: Example Search
enabled: true
version: 1

match:
  hosts:
    - example.com

entry:
  mode: url
  baseUrl: "https://example.com"

search:
  url: "https://example.com/search?q={{keyword}}"

detect:
  - id: captcha
    severity: pause_for_user
    any:
      - selectorExists: ".captcha"
      - textContains: "请完成验证"
    message: "需要人工完成验证。"

results:
  passes:
    - id: main

      containers:
        - "article.result"

      fields:
        title:
          selectors:
            - "h2 a"
          extract:
            source: text

        url:
          selectors:
            - "h2 a"
          extract:
            source: href

        snippet:
          selectors:
            - ".summary"
          extract:
            source: text
          optional: true

pagination:
  type: click

  next:
    selectors:
      - "a.next"

  maxPages: 5

delays:
  page:
    minMs: 1000
    maxMs: 2000

  turnPage:
    minMs: 1200
    maxMs: 2500

limits:
  maxResults: 200
  timeoutMs: 120000
```

---

# 38. Site Rule

`type: site`

用于小说站、内容站等详情页解析。

目标：

> 从已经人工确认的候选作品页中提取作品信息、目录和章节链接。

基础结构：

```yaml
schemaVersion: 1
type: site

id: example-novel-site
name: Example Novel Site

match:
  hosts:
    - novel.example.com

book: {}
catalog: {}
chapter: {}
limits: {}
```

---

# 39. Site Book

```yaml
book:
  title:
    selectors:
      - "h1.book-title"
    extract:
      source: text

  author:
    selectors:
      - ".author"
    extract:
      source: text
    optional: true
```

---

# 40. Site Catalog

```yaml
catalog:
  entry:
    selectors:
      - "a.catalog"

  chapterLinks:
    containers:
      - "#chapter-list li"

    fields:
      title:
        selectors:
          - "a"
        extract:
          source: text

      url:
        selectors:
          - "a"
        extract:
          source: href

  pagination:
    type: click
    next:
      selectors:
        - ".next"
    maxPages: 20
```

如果作品页本身就是目录，可以省略 `entry`。

---

# 41. Site Chapter

```yaml
chapter:
  title:
    selectors:
      - "h1"
    extract:
      source: text

  content:
    selectors:
      - "#content"
      - ".chapter-content"
    extract:
      source: text
```

v0.1 不要求全文保存。

正文主要用于：

- 页面判断；
- 可选 AI 辅助；
- 证据摘要。

必须遵守应用的数据最小化原则。

---

# 42. Site Limits

```yaml
limits:
  maxCatalogPages: 30
  maxChapters: 1000
```

必须有上限。

---

# 43. Complaint Rule

`type: complaint`

用于投诉页面辅助填写。

基础结构：

```yaml
schemaVersion: 1
type: complaint

id: example-platform
name: Example Platform

entry:
  url: "https://example.com/copyright"

requires: []

navigation: []

fields: {}

uploads: {}

batch: {}

submit:
  mode: user-approved
  finalAction: adapter
```

---

# 44. Complaint Requires

```yaml
requires:
  - work.title
  - work.officialUrl
  - author.realName
  - author.contact
  - complaint.urls
```

`requires` 只是材料声明。

不意味着 Agent 可以自由读取所有材料。

敏感字段读取仍受 AGENTS.md 隐私规则限制。

---

# 45. Complaint Fields

```yaml
fields:
  realName:
    selector: "#real-name"
    source: author.realName

  workTitle:
    selector: "#work-title"
    source: work.title

  officialUrl:
    selector: "#official-url"
    source: work.officialUrl

  infringementUrls:
    selector: "#illegal-urls"
    source: complaint.urls
    format: newline
```

格式支持：

```text
plain
newline
comma
json
```

v0.1 不允许 arbitrary template script。

---

# 46. Complaint Uploads

```yaml
uploads:
  identity:
    selector: "input[name=identity]"
    source: author.identityDocument

  copyrightProof:
    selector: "input[name=proof]"
    source: work.copyrightProof
```

文件只从用户本地读取。

真实身份或权利材料在被读取并注入第三方页面/上传控件之前，必须已经获得针对当前接收方和用途的用户授权。读取本地文件用于用户已确认的官方投诉，不等于允许把该文件交给 Coding AI、外部模型或维修工作区。

---

# 47. Complaint Batch

```yaml
batch:
  maxUrls: 50
```

如果平台限制一次只能提交 N 条：

应用层负责拆分任务。

---

# 48. Complaint Submit

必须：

```yaml
submit:
  mode: user-approved
  finalAction: adapter
```

v0.1 不支持任何其他值。

最终提交必须人工确认。

---

# 49. Panel Rule

`type: panel`

用于微博搜索、特殊内容面板等非标准搜索页。

基础：

```yaml
schemaVersion: 1
type: panel

id: example-panel
name: Example Panel

homeUrl: "https://example.com"

partition: "example"

urlGuards: []

scraper: {}

actions: []
```

---

# 50. URL Guards

```yaml
urlGuards:
  - when:
      urlMatches: "^https://example.com/login"

    action:
      type: redirect
      url: "https://example.com/search"
```

只允许白名单动作。

v0.1：

```text
redirect
block
pause_for_user
```

---

# 51. Panel Scraper

```yaml
scraper:
  containers:
    - ".feed-item"
    - "article"

  fields:
    id:
      selectors:
        - "[data-id]"
      extract:
        source: attr
        name: data-id

    userName:
      selectors:
        - ".user-name"
      extract:
        source: text
      maxLen: 30

    content:
      selectors:
        - ".content"
      extract:
        source: text
      maxLen: 200

  deduplicateBy: id

  intervalMs: 3000

  viewportOnly: true
```

---

# 52. Panel Actions

Action 只能描述用户可执行的功能入口。

例如：

```yaml
actions:
  - type: collect
    label: 收集

  - type: report
    label: 投诉
    navigate:
      url: "https://example.com/report?id={{item.id}}"
```

任何具有外部副作用的 action 都不得默认自动执行。

---

# 53. Rule Runtime Context

规则执行器可以读取有限上下文：

```text
keyword
page
offset
work
item
currentUrl
runtimeVariables
```

不得自动暴露：

- API Key
- Cookie
- Token
- 整个数据库
- 用户私有目录
- 浏览器 Profile

---

# 54. 标准 SearchResult

所有 search 规则最终映射到：

```ts
interface SearchResult {
  title: string
  url: string
  snippet?: string
  sourceRuleId: string
  keyword: string
  page?: number
  discoveredAt: string
  metadata?: Record<string, string | number | boolean | null>
}
```

---

# 55. 标准 ChapterLink

```ts
interface ChapterLink {
  title?: string
  url: string
  index?: number
}
```

---

# 56. 标准 Diagnostic

```ts
interface RuleDiagnostic {
  ruleId: string
  ruleType: string
  stage: string
  message: string
  url?: string
  detectionId?: string
  expected?: unknown
  actual?: unknown
  timestamp: string
}
```

进入 Harness 工作区之前必须脱敏。

---

# 57. 规则验证

每条规则至少执行：

```text
Schema Validation
Fixture Validation
```

发布前推荐：

```text
Targeted Live Smoke Test
```

---

# 58. Fixture 结构

建议：

```text
tests/fixtures/
  search/
    bing/
      result-page.html
      expected.json

  sites/
    example-site/
      book.html
      catalog.html
      expected.json

  complaint/
    example-platform/
      form.html
```

Fixture 不得含真实用户敏感信息。

---

# 59. Expected Fixture

搜索：

```json
{
  "minResults": 3,
  "requiredFields": [
    "title",
    "url"
  ]
}
```

站点：

```json
{
  "bookTitle": "示例作品",
  "minChapterLinks": 10
}
```

---

# 60. Repair Compatibility

为了支持“用 AI 修复规则”，每条规则应尽量满足：

- 单文件可理解；
- 单文件可替换；
- 依赖明确；
- Fixture 可独立运行；
- 不依赖隐藏代码；
- 不依赖未文档化远程数据。

维修 Agent 默认只修改：

```text
candidate.yaml
```

---

# 61. Repair Result

维修 Agent 必须输出：

```json
{
  "status": "success",
  "ruleId": "example",
  "schemaVersion": 1,
  "ruleVersionBefore": 4,
  "ruleVersionAfter": 5,
  "changedFields": [
    "results.passes[0].containers",
    "results.passes[0].fields.url"
  ],
  "tests": {
    "schema": "pass",
    "fixture": "pass",
    "targeted": "pass",
    "live": "pass"
  },
  "coreCodeChanged": false,
  "summary": "..."
}
```

---

# 62. 规则应用前检查

正式应用 candidate 前必须验证：

1. schemaVersion 支持；
2. rule id 未变化；
3. rule type 未变化；
4. 不包含未知 Step；
5. 不包含任意脚本；
6. 不包含绝对本地路径；
7. 不包含 secret；
8. 测试达到应用策略要求。

---

# 63. 原子替换

规则应用：

```text
candidate validate
↓
backup current
↓
atomic replace
↓
reload
↓
smoke test
↓
success / rollback
```

---

# 64. 安全禁止项

规则不得包含：

- JavaScript
- shell command
- executable path
- eval
- Function constructor
- arbitrary HTTP POST body to unknown service
- API Key
- Cookie
- Token
- 用户真实身份证号码
- 用户真实私人数据
- 验证码识别/绕过脚本
- 浏览器指纹伪装配置
- 绕过访问控制逻辑

---

# 65. 网络边界

规则中声明的 host 必须参与 allowlist 校验。

例如 search 规则：

```yaml
network:
  allowedHosts:
    - example.com
    - www.example.com
```

如果 Step 导航到未声明 host：

```text
拒绝或请求用户确认
```

不得静默访问未知第三方。

---

# 66. 文件边界

规则不能声明任意磁盘路径。

允许：

```text
runtime screenshot
runtime fixture
runtime diagnostic
user-selected upload
```

禁止：

```text
C:\Users\...
/home/user/...
../../secret
```

---

# 67. Live Test 原则

Live Test 只验证规则是否仍可正常解析。

不应：

- 大规模爬取；
- 批量翻页；
- 自动投诉；
- 自动提交；
- 绕验证码。

推荐：

```text
1 个安全测试关键词
1 页
少量结果
```

---

# 68. Harness 自动生成规则要求

当 DeepSeek Harness 新增或修复规则时：

1. 必须先阅读 `AGENTS.md`；
2. 必须阅读本文件；
3. 优先使用稳定 selector；
4. 避免深层 nth-child；
5. 生成 fixture；
6. 生成 expected；
7. 运行 schema test；
8. 运行 fixture test；
9. 可行时运行 live smoke test；
10. 输出 repair-result.json；
11. 不修改正式规则；
12. 不修改测试以伪造成功。

---

# 69. Schema 扩展原则

如果 Agent 发现当前协议无法描述网站行为：

不得直接写站点专属核心代码。

应该先提出：

```text
需要新增什么通用原语？
为什么现有原语无法表达？
是否其他网站也可能使用？
安全边界是什么？
如何测试？
```

只有通用价值成立后，才扩展协议。

---

# 70. v0.1 明确不支持

为了保持协议可控，v0.1 暂不支持：

- 任意 JavaScript 执行
- 用户自定义代码注入
- 复杂表达式语言
- 自定义 shell
- 自动验证码识别
- 浏览器反检测脚本
- 任意网络请求 DSL
- 未经本次用户确认的自动最终投诉
- 任意远程规则脚本
- 自动远程规则更新
- 规则包加密执行

---

# 71. 兼容与迁移

未来：

```yaml
schemaVersion: 2
```

必须提供 migration。

旧规则不得静默失效。

Migration 应：

- 保留原规则备份；
- 输出迁移摘要；
- 运行测试；
- 失败自动回滚。

---

# 72. 推荐的第一批规则

用于验证 v0.1：

```text
1. 一个普通搜索引擎
2. 一个需要 interaction 的网盘搜索站
3. 一个小说目录站
4. 一个简单投诉表单 fixture
```

不要一开始追求覆盖大量网站。

目标是验证：

> 这套协议是否足够让 Harness 在不修改核心代码的情况下完成适配与修复。

---

# 73. v0.1 验收标准

RULES_SPEC v0.1 可以进入下一阶段，至少需要通过以下实验：

## 实验 A：普通搜索

Harness 能生成一条 search rule，并正确提取：

- title
- url
- snippet

## 实验 B：二次链接解析

Harness 能用 Step DSL 表达：

```text
点击结果
↓
进入/打开新页面
↓
提取真实 URL
↓
返回列表
```

## 实验 C：分页

至少验证：

- click 或 url pagination

## 实验 D：目录提取

Harness 能生成 site rule：

```text
作品页
↓
目录
↓
章节链接
```

## 实验 E：规则修复

人为修改 fixture 使旧 selector 失效。

Harness 必须：

1. 根据诊断找到问题；
2. 只修改 candidate；
3. 通过测试；
4. 不修改核心程序。

---

# 74. 最终设计原则

本协议不是为了把所有网站的所有可能行为都塞进 YAML。

它的目标是覆盖：

> **高频、可重复、可审计、适合由 Agent 自动维护的网页适配行为。**

当网站复杂到声明式规则无法合理表达时：

优先考虑：

1. 是否能扩展通用原语；
2. 是否值得增加通用 Adapter；
3. 是否应该保留人工步骤。

而不是立即加入任意脚本能力。

本项目宁可保留少量人工步骤，也不应为了“完全自动化”牺牲：

- 隐私
- 安全
- 可测试性
- 可解释性
- 可维护性

---

# 75. 一句话总结

> **规则描述网站，执行器解释规则，测试证明规则，Harness 修复规则，用户决定是否投诉；用户确认后，软件可以替她完成机械提交。**
