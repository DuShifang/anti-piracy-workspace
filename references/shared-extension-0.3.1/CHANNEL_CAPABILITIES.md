# 渠道能力 0.3.1

所有下列渠道已进入生产扩展的平台流程执行器；fill=true、upload=true、submit=false。能力限于所列渠道的具体业务流程，不能外推为平台所有权利类型或其他业务都支持。当前流程主要为个人作者的文字/小说著作权；主体、证件类型和其他业务选项请在动作预览中核对。

| channelKey | 渠道 / 子产品 | 适配模块 | 本扩展每批对象限额 |
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

限额为本扩展的任务分批策略，各渠道采用上表所列限额，不能当作各官网最新官方上限。网页限制更严格时以网页为准并暂停。prepareBatches 为每批重新生成链接文件，不能把全量链接文件重复用于所有批次。不同批次须重新配对并核对。

## 渠道所用资料

除 work.title、work.officialUrl、objects、body、identity 基本信息外，按当前实际步骤补充下表。非当前步骤可暂缺；执行到缺少项时暂停，可本人处理，或回软件补齐、增加 revision、重新交接。自动生成不替代证明文件的真实性与充分性。

| 渠道 | 额外字段 | 使用的材料 role | 特殊行为 |
|---|---|---|---|
| 夸克 / UC 实名 | identity.idCard | id_front、id_back | 本人确认实名；不采集登录凭据 |
| 夸克网盘/搜索、UC网盘/神马 | context.holderLabel（默认姓名）；网盘 objects[].accessCode | authorization、重复 contract、screenshot_backend、screenshot_summary、对应 complaintXlsx_quark/shenma | 模块值由 channelKey 固定；权利人按标签唯一匹配；导入 Excel 后本人复核解析结果 |
| 百度主体资料 | identity.idCard | id_front、id_back、id_hold | 锁定档案跳过填写并自动进入下一步；验证码本人操作 |
| 百度权属选择/登记 | work.authorName | screenshot_backend、screenshot_summary、全部 contract、authorization | 书名与审核状态精确匹配；长期有效声明本人确认 |
| 百度投诉 | work.authorName、context.baiduService | 不依赖 Excel | 逐行填写，按明确重复/已投诉错误处理；不删除其他报错行；自动触发链接校验并处理分页；扫码验证及最终提交本人操作 |
| Bing | identity.firstName、lastName、country；work.summary；identity.organization 按实际填写 | copyright_proof_pdf、complaintTxt_bing | Ebook 分支；电子签名、三项法律声明、验证码、提交本人操作 |
| 迅雷 | identity.idCard、address；work.rightsStart、rightsEnd；context.rightsAcquisition、trusteePresent | id_front、id_back；contract/authorization/authorization_stamp/screenshot_backend/screenshot_summary 合计最多10；complaintXlsx_xunlei | 云盘分支；权利起止日期由任务明确提供；800字正文限制 |
| 微博 | identity.idCard | id_front、id_back、copyright_proof_docx（该用途可为 PDF） | Word/PDF 权属槽；声明与验证码本人操作 |
| 微信公众号 | identity 基本信息 | id_merged（单文件≤5MiB）；category=evidence 且 disclosure=may_forward 的证据 | 正文≤150字不截断；按 URL 逐个添加；证据可能转交被投诉方，身份和权利证明不能放该槽 |
| 淘天/闲鱼实名 | identity.idCard | id_front、id_back | 承诺函下载、签字、扫描上传及扫码本人处理 |
| 淘天权属登记 | work.registrationDate、publishDate；identity.organization、context.brandLabel 仅商品化分支 | contract/authorization_stamp；screenshot_summary/detail；screenshot_release | 材料类型、署名事实、商品化事实本人确认，登记日期和企业信息来自任务资料 |
| 闲鱼投诉 | context.copyrightWorkLabel（下拉显示的完整标签） | screenshot_complaint，最多4份，每份≤20MiB | 支持投诉截图与链接输入；验证链接与最终提交本人操作 |
| 晋江绿色通道 | work.novelId、context.quarkSearchUrl；验证时 identity.idCard | screenshot_quark_search；证件路径 id_both | 适用于晋江绿色通道中的夸克小说维权分支；身份验证、证件/截图确认本人操作 |
| 晋江维权证件 | 无额外文本 | id_both | 展开证件上传；确认提交本人操作；拼图不自动打码 |
| 百度智能体 | identity.idCard | screenshot_complaint；screenshot_backend 必须为可转发 evidence；id_front、id_back | 每批单 URL；使用当前页面的动态上传控件；1000字限制 |
| QQ邮箱 | context.mailRecipients：实际投诉收件人列表 | 明确交接的附件；自动生成 complaintDocx_360 可附本人提供 signature | 正文/附件/主题/收件人草稿；不点击发送；不要将与此收件人无关的材料交接为邮件附件 |

body 是作者软件准备的完整、已复核、适合该渠道长度的正文。facts 和 request 在本地预览并用于生成邮件 Word；网页正文通常直接使用 body，不偷偷拼接、改写或截断。百度≤200、夸克/UC≤1000、微信≤150、迅雷≤800；其他输入再受网页自身 maxlength/校验约束。

## 材料组件

- linkSpreadsheet：百度三列「序号/链接名称/链接地址」、迅雷「链接名称/链接地址」、搜索「侵权链接/原创链接/作品名（选填）」。单元格为文字，避免把作品名中的 = 当公式。
- 网盘 Excel 第二列为「访问码」，按网盘字段准备材料。本版未现场核验官网模板表头，用户需核对导入结果；不应把文件列表出现当作行解析完成。可提供由官网模板生成的同 role 文件替换自动生成文件。
- mergeProofPdf：PNG/JPEG/PDF 合并；splitContractPdf：多页 PDF 拆为逐页 PDF（不谎称已转成图片）。图片专用槽请提供实际 PNG/JPEG。
- mergeIdentity：正反面纵向拼图，不伪造签名、水印或打码，不上传云端；需要打码的版本由作者在本地准备。
- proofDocx：中文正文加 PNG/JPEG 证明图或本人签名图。微博默认生成保留原 PDF 页的 PDF，使用同一 copyright_proof_docx 用途标识，不强行把 PDF 冒充 DOCX。
- QQ complaintDocx_360 是可编辑的通用投诉资料 Word，包含任务事实、正文、诉求、链接及提供的签名图；未冒充某部门盖章表格或保证所有收件方接受。若接收方有指定表格，提供同 role 的实际成品文件替代自动生成。

## 结果边界

文件选入可能已即时上传。Bing 的待提交文件控件以 MEMBER_FILE_SELECTED 记录文件选择核对，不代表已上传。其他材料槽在观察到新完成标记或可靠缩略图后记录 MEMBER_ATTACHMENT_OBSERVED；没有标记则停下来请本人核对，不盲等后当成功，不自动重传。上传完成以可靠状态标记或本人核对为准，不以固定等待时间判定成功。

当前页面刷新、文档或步骤切换、任务变化、产品或已绑定权利人改变，会停止旧授权。真实最终提交、邮箱发送、真实回执和投诉进度采集均不自动执行。完成整步只是 MEMBER_STEP_DONE，不是平台受理。

本版不包含抖音、腾讯文曦、小说搜索采集、投诉后下架监控。微信公众号和 QQ邮箱不能统称“所有腾讯业务”；百度智能体与百度版权中心也为不同渠道。
