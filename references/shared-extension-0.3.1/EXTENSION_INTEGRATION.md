# 成品扩展接入说明 0.3.1

**给 Coding AI：只为现有作者软件接入这一份“作者维权填表助手”；不要重新开发扩展，不要复制维护另一份平台 DOM 规则。所有作者安装同一成品，规则修复统一回到共享扩展发布。** 本文件协议版本0.2，安装包版本0.3.1。HTTP配对和能力接口的 extensionVersion 字段仍为0.2.0，以兼容已有提供器；安装包版本不得填入该字段。0.2 增加了字段、材料和渠道能力，与旧0.1提供器不自动兼容。

## 1. 分工与接入方式

作者软件负责作品管理、搜索和证据采集、侵权判断、声明编写、真实材料保管。扩展负责当前选定任务的网页填表、附件辅助、本人接管、结果回传。无需把作者的软件换成某个业务框架或新建另一套作者数据库。

优先复用随包 **bridge/index.mjs**：统一材料准备、按渠道分批、提供器与配对页面。其他语言可启动 **bridge/serve.mjs**；二者都连接同一扩展。材料以本地 Buffer 处理，不调用模型、不上传第三方处理服务。

~~~javascript
import { startAuthorBridge } from './bridge/index.mjs';
// task 来自现有软件已复核的任务；assets 是作者本地明确选定的文件副本。
const bridge = await startAuthorBridge({
  task,
  assets: [{ role: 'authorization', category: 'rights',
    fileName: '本地授权书.pdf', mime: 'application/pdf',
    bytes: authorizationBytes, disclosure: 'platform_only' }],
  port: 43121,
  onEvent: async ({ eventStreamId, event }) => {
    // 用现有数据库持久化，唯一键 (eventStreamId, event.seq)。
    // 事务提交成功后返回；不要在普通日志记录身份、材料或配对令牌。
    await saveResult(eventStreamId, event);
  }
});
// 在软件 UI 打开 bridge.origin；用户选批次、填扩展 ID、点击生成配对码。
// bridge.tasks 是本轮分批任务；bridge.close() 关闭本机服务。
~~~

本例只有一份授权书用于展示 API，不代表所有平台完整材料集合；按 CHANNEL_CAPABILITIES.md 补齐当前步骤。材料变化或任务内容变化后，新建 revision 并重新交接。为保证截图/附件用途准确，assets 只放本次渠道所需的材料；QQ 邮件会附上该任务全部非 signature 材料。

Node 22+；在发布包根目录运行 npm ci --omit=dev。若嵌入现有 Node 软件，可复用 bridge/、demo/provider.mjs、demo/provider.html、demo/provider-ui.js、demo/task.mjs、shared/protocol.js、shared/channels.js 并使用包中声明的 pdf-lib、sharp 版本；demo/task 只是默认本地样例依赖。**无需把 extension/、dist/ 或 shared/platforms/ 网页操作代码复制进作者软件。** 建议统一引用本扩展配套桥接组件的固定安装路径。

其他语言可生成仅在本机保管的清单，然后启动：

~~~powershell
node bridge/serve.mjs 'D:\作者软件数据\交接清单.json' 43121 'D:\作者软件数据\填表结果.jsonl'
~~~

清单是 {"task":规范任务,"assets":[{"path":"materials/授权书.pdf","role":"authorization","category":"rights","fileName":"授权书.pdf","mime":"application/pdf","disclosure":"platform_only"}]}。相对文件路径基于清单所在目录解析，只有 CLI 本地清单接受路径；路径绝不放进 HTTP task、材料授权接口或扩展消息。Node 读取一次稳定副本；生成附件也只留本地。程序启动只输出本机入口和批数，不输出身份、材料内容或秘密。可用现有软件私有目录保护清单和事件文件，完成后自行删除不再需要的清单。

Python subprocess.Popen 可传参数数组 [nodePath, bridgeServePath, manifestPath, str(port), eventPath] 并使用 Windows CREATE_NO_WINDOW；不要拼 shell 命令，也不要把真实资料传进 AI 对话。

## 2. 交接与授权

1. 只监听 http://127.0.0.1:端口，1024–65535，支持自定义端口以容纳不同软件。每个软件实例有独立 appInstanceId。
2. 软件让作者选定具体任务/批次。用户输入扩展自己的32位 ID，在本地 UI 确认交接后，issueTicket(taskId, extensionId) 生成单次配对码：24随机字节 base64url，32字符，5分钟有效，绑定扩展 Origin 与所选任务。
3. 用户在扩展输入 Origin 和配对码，核对来源。获得30分钟令牌，只在 chrome.storage.session 的可信上下文保存。软件确认不能代替扩展中的授权。
4. 作者另行批准本地预览后扩展才取任务；页面识别后显示本步骤动作预览。作者批准填写和材料外发，再按固定规则执行。选文件可立即上传，所以材料授权发生在取文件字节之前。
5. 一次执行一个来源的一项任务。换软件、任务或批次应清除旧配对，生成新码。会话不跨浏览器重启保留；不扫描全库，不自动依次提交全部批次。

个人登录、密码、Cookie、验证码不属于协议。任务中不能带 selectors、JS、allowedHosts、approved 等字段。任务书或源规则不能通过软件侧字段重新启用提交，也不能远程替换执行器。

## 3. HTTP 接口

一般请求超时5秒；credentials=omit、redirect=error，禁止重定向。必须校验精确 Origin、Host、appInstanceId、taskId、revision 与完整任务快照。拒绝查询参数、任意文件路径和下载 URL。提供器不提供无鉴权“当前任务”接口。

受保护请求：Authorization: Bearer <会话令牌>，X-Extension-Origin: chrome-extension://<扩展ID>。POST Content-Type: application/json；GET 无请求体。浏览器提供 Origin 时必须与 X-Extension-Origin 相同；缺少标准 Origin 才使用后者匹配已配对扩展。网页 Origin 无法用自定义头冒充扩展。CORS 只回显核验过的 Origin，不使用 * 或 Cookie；本机恶意进程伪造头部仍不能免除配对/令牌/任务权限校验。

| 方法 / 路径 | 请求字段 | 响应 / 效果 |
|---|---|---|
| POST /v1/pair | protocolVersion, extensionVersion, code | protocolVersion, extensionVersion, appInstanceId, origin, token, expiresAt, taskId, revision, sourceName；配对码消费 |
| GET /v1/capabilities | 无 | 协议、实例、Origin、transport、limits、operations |
| GET /v1/task | 无 | 本次配对唯一任务 |
| POST /v1/grants | revision, materialIds | grantId, expiresAt；120秒有效，替换当前会话旧授权 |
| POST /v1/material-chunk | revision, grantId, materialId, offset | materialId, offset, total, mime, data, done |
| POST /v1/events | 第6节完整事件 | acceptedSeq, duplicate |
| POST /v1/cancel | {} | {"ok":true}；禁止后续任务/材料读取，仍接受事件 |
| POST /v1/revoke | {} | {"ok":true}；撤销会话 |

成功统一 HTTP200，预检204，失败 HTTP400 {"error":"稳定错误码"}；JSON未知字段拒绝。配对全服务每分钟最多10次尝试。示例的 /local/list、/local/pair、/local/revoke 为软件的同源 UI 接口，必须 POST、实际同源、带 X-Local-CSRF；它们不是扩展的另一条运输路线。自有界面可调用 issueTicket，但只应在作者明确选任务并点击交接时调用。

能力对象实际形状（appInstanceId / origin 按实例替换）：

~~~json
{
  "protocolVersion": "0.2",
  "extensionVersion": "0.2.0",
  "appInstanceId": "example-instance",
  "origin": "http://127.0.0.1:43121",
  "transport": "loopback-http",
  "limits": {
    "fileBytes": 20971520,
    "totalBytes": 104857600,
    "chunkBytes": 49152,
    "materials": 64,
    "objects": 1000,
    "timeoutMs": 5000
  },
  "operations": [
    "preview",
    "materialGrant",
    "materialChunk",
    "events",
    "cancel",
    "revoke"
  ]
}
~~~

扩展逐项校验该能力对象，本版只接受协议0.2与HTTP extensionVersion=0.2.0；浏览器安装包版本为0.3.1。这是提供器接口能力，渠道能力来自扩展包，不由软件声明覆盖。

## 4. 规范任务

必填根字段：protocolVersion、appInstanceId、taskId、revision、expiresAt、sourceName、channelKey、work、objects、facts、body、request、identity、materials、requestedOperations；可选 context。字段类型和限制以 shared/protocol.js 为准。

以下是纯虚构结构示例，仅用于本地模拟；日期运行时改为未来。真实平台只交接有权办理且已核对的真实事项。

~~~json
{
  "protocolVersion":"0.2", "appInstanceId":"example-instance", "taskId":"task-001",
  "revision":1, "expiresAt":"2026-09-13T23:00:00.000+08:00",
  "sourceName":"本地作者软件（虚构示例）", "channelKey":"local.mock.copyright",
  "work":{"title":"虚构测试作品","officialUrl":"https://example.invalid/book/1"},
  "objects":[{"objectId":"object-a","title":"虚构对象A","url":"https://example.invalid/share/a"},
             {"objectId":"object-b","title":"虚构对象B","url":"https://example.invalid/share/b"}],
  "facts":"仅供本地模拟。", "body":"已复核的本地虚构测试正文。", "request":"核对本地模拟对象。",
  "identity":{"name":"虚构作者","email":"test@example.invalid","phone":"000-TEST-ONLY"},
  "materials":[], "requestedOperations":["fill"]
}
~~~

桥接 startAuthorBridge/prepareTask 输入中 materials 可以为空，它会由 assets 重建材料元数据并增加生成文件；local.mock.copyright 直接用 demo/server.mjs 或 createProvider，不进入真实渠道材料准备函数。

| 字段 | 类型与含义 |
|---|---|
| work.title / officialUrl | 必填作品名≤300字符、http/https正版URL（不允许用户名密码） |
| work.authorName / summary / novelId | 可选笔名、简介≤10000、纯数字作品ID |
| work.publishDate / registrationDate / rightsStart / rightsEnd | 可选有效日期 YYYY-MM-DD；开始不能晚于结束。不能把首发日期推算成固定保护年限 |
| identity.name / email / phone | 必填姓名≤100、邮箱≤254、电话≤40；软件负责真实性和格式 |
| identity.idCard / address / firstName / lastName / organization / country | 可选，各≤500；Bing country 是页面实际选项值（如 China），organization 无则本人处理可选栏 |
| objects[] | 每条 objectId、title、url 必填；可选 accessCode（0–100字符）用于网盘；objectId 唯一 |
| facts / body / request | 必填，分别≤10000/20000/3000。body 原样入表，需预先满足渠道更小上限 |
| context.holderLabel | 权利人下拉精确标签，省略默认 identity.name；不会任选第一项 |
| context.copyrightWorkLabel / brandLabel | 淘天已登记作品的完整选项、适用时的品牌标签 |
| context.baiduService / quarkSearchUrl | 百度版权中心服务标签 / 晋江通道所需夸克搜索结果URL |
| context.rightsAcquisition / trusteePresent | 迅雷实际权利取得选项文本、是否有委托人 boolean；没有值会暂停而非默认 |
| context.mailRecipients | 1–10个明确邮箱；不猜测部门/平台地址 |

ID 为1–80位字母数字下划线短横线；revision 为正安全整数。单个规范任务1–1000对象、最多64材料，材料单份≤20MiB、合计≤100MiB；最低1字节。单个渠道还有第7节分批限额。正文、元数据、字节或材料用途变化需增加 revision；字节变化使用新 materialId。每个 ID 对应不可变字节副本，不把 ID 映射为可随时被覆盖的磁盘路径。

requestedOperations 只能是 fill、upload、mockSubmit 的不重复数组；真实渠道通常 ["fill","upload"]，只填文字可 ["fill"]。这只是请求，不代表用户已经授权。mockSubmit 仅本地演示包和指定模拟页有效。

## 5. 材料、生成与分块

规范材料必填 materialId、category、fileName、mime、size；可选 role、disclosure（真实规则按 role 查找，因此必须正确提供）。category 为 identity、rights、evidence、links。fileName≤100字符，无斜杠/反斜杠/控制字符。MIME允许 TXT、PDF、PNG、JPEG、XLSX、DOCX，具体网页 input.accept 和槽位上限更严格时暂停。

全部 role：

`contract`、`authorization`、`authorization_stamp`、`delete_guarantee`、`copyright_proof_pdf`、`copyright_proof_docx`、`id_front`、`id_back`、`id_hold`、`id_both`、`id_merged`、`screenshot_backend`、`screenshot_summary`、`screenshot_quark_search`、`screenshot_complaint`、`screenshot_detail`、`screenshot_release`、`signature`、`complaintDocx_360`、`complaintTxt_bing`、`complaintXlsx_baidu`、`complaintXlsx_quark`、`complaintXlsx_shenma`、`complaintXlsx_xunlei`。

同 role 可多份的用途有 contract、重复证据等，按 assets 顺序处理；其余单文件角色应只提供一份。扩展不会自动猜同 role 中哪个文件正确。id_* 和 signature 的 category 必须为 identity。disclosure 默认 platform_only；may_forward 表示已经明确知道证据可能转交被投诉方。微信补充材料/百度智能体可转发槽必须 category=evidence 且 disclosure=may_forward；身份文件永不进该槽。

资产输入：{role,category,fileName,mime,bytes:Buffer,disclosure?}。本地共享组件暴露：

| 导出方法 | 用途 |
|---|---|
| prepareTask(task, assets) → {task, files:Map} | 规范材料ID、生成该渠道所需链接表/TXT/证明PDF/Word/证件拼图 |
| prepareBatches(task, assets) → 数组 | 按渠道分批、每批新任务ID和独立链接文件；不丢对象 |
| linkSpreadsheet(task) → Buffer | 生成 XLSX；百度当前执行器使用逐行填写，导出表可供软件另存 |
| mergeProofPdf(assets) → Promise<Buffer> | PNG/JPEG/PDF 合并为 PDF |
| splitContractPdf(bytes) → Promise<Buffer[]> | 合同PDF拆页；输出仍为PDF |
| mergeIdentity(frontBytes, backBytes) → Promise<Buffer> | 证件正反纵向拼成 JPEG；不自动打码 |
| proofDocx(text, imageAssets?) → Promise<Buffer> | 中文文字与 PNG/JPEG 图片生成 Word |

prepareTask 仅在同 role 成品不存在时生成，可用作者本地审核过的官网指定成品替换。生成只做格式处理，不创造权属、签名或事实。Bing/微博 PDF 需要实际证明材料；QQ Word 未提供 signature 时不生成假签名。格式、网盘表头的已知限制见 CHANNEL_CAPABILITIES.md。

多批任务如果传入预制 complaintXlsx_* / complaintTxt_* 全量文件会报 BATCH_PREBUILT_LINKS，防止第二批又带第一批链接。让 prepareBatches 生成，或软件先分批、为每批独立调用 prepareTask。

授权/分块示例：

~~~json
{"revision":1,"materialIds":["material-1"]}
{"revision":1,"grantId":"授权返回值","materialId":"material-1","offset":0}
{"materialId":"material-1","offset":0,"total":100,"mime":"text/plain","data":"base64内容","done":true}
~~~

扩展仅为当前附件动作请求独立材料授权，单块原始字节≤49152；严格顺序 offset，从0开始，不跳块/重复读，已完成材料不可复用原授权重读。后台校验 materialId、total、mime、偏移和 done，只将当前块交给目标文档的隔离脚本；会话令牌与 grantId 不进入网页。

页面隔离内存组装 File/DataTransfer，触发 input/change；文件字节不写扩展存储。临时缓冲在选择文件、取消或60秒空闲时清除。已有不属于本次动作的文件或未知上传状态会暂停。材料槽串行操作，选中与上传确认分开。Bing 的待提交文件控件记录 MEMBER_FILE_SELECTED，表示文件选择已核对；其他材料槽按可观察状态记录 MEMBER_ATTACHMENT_OBSERVED，15秒无可靠完成状态需本人核对，不能自动重传。两类事件均不代表投诉已提交或受理。

## 6. 事件与恢复

~~~json
{"protocolVersion":"0.2","appInstanceId":"example-instance","taskId":"task-001","revision":1,
 "seq":4,"status":"ready","code":"MEMBER_ADVANCED","completed":["r3_s1_f2"],
 "submission":"not_submitted","receipt":null}
~~~

seq 从每次配对的1严格递增；同 seq 同内容返回 duplicate=true；同 seq 不同内容 EVENT_CONFLICT；跳序 EVENT_ORDER。用 (eventStreamId, seq) 唯一键持久化；eventStreamId 是提供器私有回调的随机事件流ID，不是令牌，也不加入规范 HTTP 事件。onEvent 提交成功才确认；如回调结果不确定，重试可能再次进入回调，数据库按该唯一键去重。

completed 是动作/字段/材料ID数组（最多12000项，各1–100位字母数字下划线短横线）。重复展开添加 _iN；本人处理标记 _manual；明确重复链接返回 objectId_duplicate。不要用它推断投诉成功。进程内循环会返回动作完成，但本人接管与控件确认仍按 code 区分。

| status / code | 软件应显示 |
|---|---|
| previewed / PREVIEWED | 已批准本地预览 |
| ready / MEMBER_APPROVED、MEMBER_ADVANCED | 当前步骤获准、动作已推进 |
| bytes_received / BYTES_RECEIVED | 材料字节到页面缓冲 |
| files_selected / FILES_SELECTED | 控件选中文件，可能已即时上传 |
| ready / MEMBER_UPLOAD_CONFIRMED | 当前槽可靠完成标记；不是投诉受理 |
| needs_user / MEMBER_MANUAL、MEMBER_UPLOAD_CONSENT | 需要本人操作或附件授权 |
| needs_user / MEMBER_FILE_CONFIRM | 未取得可靠上传确认，由本人核对 |
| ready / MANUAL_ACKNOWLEDGED | 仅记录本人确认，不等于程序验证完成 |
| needs_user / MEMBER_STEP_DONE | 当前辅助步骤结束；本人复核并推进/提交 |
| partial、paused | 部分执行或暂停，保留已完成项，不能自动重放 |
| cancelled、cleared | 停止后续/解除配对；不能撤回已外发 |
| text_checked、upload_confirmed | 本地模拟的控件检查/附件确认状态 |
| submission_unknown、submitted、rejected | 当前只用于本地模拟提交 |

submission 独立为 not_submitted、unknown、submitted、rejected。本版真实渠道恒不自动提交；not_submitted 表示扩展未执行自动提交，**不代表作者没有在网页手动提交**。receipt 当前只允许 {"id":"MOCK-1","scope":"local_mock_only"}，真实回执不在本版自动采集范围。软件可单独让作者填写真实回执，但不能标为扩展观察结果。

每个动作绑定 tabId、documentId、URL、步骤、task/revision 和30分钟内的当前授权。页面刷新/换步骤、产品或绑定权利人变更、断线、任务变化、过期均停止。恢复可继续未完成动作；结果不明时请本人处理当前项并确认，避免重复上传/新增行。变更任务需新的 revision 和重新配对，不允许用后台推送覆盖用户已批准内容。

Worker 中断检测到已持久化的在途动作标记时进入 WORKER_INTERRUPTED，不自动重放；未确认事件保留在 session 按序重发。浏览器重启后 session 丢失，软件应显示“执行中断，需本人核对”，不能推断没有外发。诊断只导出版本、状态、代码、渠道和待回传数量。

常见错误：TASK_CHANGED/APPROVAL_EXPIRED/PAGE_CHANGED 重新交接或识别；MISSING_VARIABLE/MATERIAL_MISSING 回软件补充；USER_EDIT_CONFLICT/FILES_ALREADY_PRESENT 保留网页已有内容；MATERIAL_DISCLOSURE 纠正材料用途；WAIT_TIMEOUT/CONTROL_REJECTED/UPLOAD_REJECTED/VALIDATION_REQUIRES_USER 本人核对或反馈改版；SUBMIT_DISABLED 不尝试绕开。

## 7. 已开放渠道

| channelKey | 渠道 / 子产品 | 规则组 | 本扩展每批对象限额 |
|---|---|---|---|
| `quark.pan.copyright` | 夸克网盘 | ipp.quark.cn | 500 |
| `quark.search.copyright` | 夸克搜索 | ipp.quark.cn | 500 |
| `uc.pan.copyright` | UC网盘 | ipp.uc.cn | 500 |
| `uc.search.copyright` | 神马搜索 | ipp.uc.cn | 500 |
| `baidu.copyright` | 百度版权投诉 | newcopyright.baidu.com | 200 |
| `bing.search.copyright` | Bing版权投诉 | bing.com | 1000 |
| `xunlei.pan.copyright` | 迅雷网盘 | copyright.xunlei.com | 500 |
| `weibo.copyright` | 微博知识产权 | service.account.weibo.com | 100 |
| `wechat.public.copyright` | 微信公众号 | mp.weixin.qq.com | 20 |
| `taobao.xianyu.copyright` | 淘天／闲鱼 | ipp.taobao.com | 300 |
| `jjwxc.green.copyright` | 晋江绿色通道 | my.jjwxc.net | 100 |
| `jjwxc.identity` | 晋江维权证件 | jjwxc_cert | 1 |
| `baidu.agent.copyright` | 百度智能体 | agent-proxy.baidu.com | 1 |
| `mail.qq.complaint` | QQ邮箱投诉草稿 | wx.mail.qq.com | 500 |

local.mock.copyright 仅 dist/demo 支持。本版提供12组流程，按网盘/搜索等业务细分为14个具体渠道，不是14个独立平台。详细字段、材料和具体业务分支看 CHANNEL_CAPABILITIES.md；页面步骤见 [平台流程参考](https://github.com/DuShifang/author-complaint-assistant/blob/main/FLOW_COVERAGE.md)。无抖音、腾讯文曦或小说搜索采集规则，不把 QQ 邮箱/微信公众号外推为这些功能。

## 8. 交给后续 Coding AI 的指令

> 为我的现有作者软件接入已安装的“作者维权填表助手 0.3.1”。完整阅读随包 EXTENSION_INTEGRATION.md、CHANNEL_CAPABILITIES.md，复用 bridge/index.mjs 或通过 bridge/serve.mjs 启动本机提供器，使用协议 0.2。只适配现有软件的作品、任务、材料、批次交接和结果保存；不要重新开发扩展，不要复制另一套 DOM 适配规则。对象和声明先在软件内复核，文件以当前任务的不透明不可变 materialId 交接。保留扩展独立预览/填写/材料外发授权，登录、验证码、法律声明、签名和真实最终提交由本人处理。真实受理不从控件填好或附件完成推断。先使用现有本地模拟站，不得向真实平台提交虚构事项，不得读取或输出 Cookie、密码、真实身份材料到 AI 对话。规则改版回到共享扩展统一修复；不用每位作者都生成一份扩展。

本版已进行本地测试、浏览器扩展模拟交接和生产执行器离线页面测试，未进行真实平台登录后实测。结果见VALIDATION_REPORT.md；官网实际使用效果由作者在真实事项中核对。
