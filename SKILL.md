---
name: wushuang-baoshui
name_en: Wushuang Auto Tax Filing
name_zh: 无双报税自动版
description: Automated quarterly tax-filing companion for Zhejiang small-scale taxpayers, paired with the local single-file "Wushuang Tax" app. Chat in natural language to auto-download invoices, build the ledger, and generate VAT / corporate-income / financial / property-behavior / individual-income reports, then produce per-form operation cards; reading is automated while form-filing and submission default to the user's own step-by-step confirmation at the e-Tax bureau. Trigger terms — 开始报税/报税/下载发票/生成财报/填报台/电子税务局操作/无双报税自动版.
description_en: Automated quarterly tax-filing companion for Zhejiang small-scale taxpayers, paired with the local single-file "Wushuang Tax" app. Chat to auto-download invoices, build the ledger, generate the five filing tables and per-form operation cards; reading is automated while filing and submission stay a manual, user-confirmed lab feature.
description_zh: 无双报税自动版（浙江·小规模纳税人）。与本地单文件软件「无双报税」联动：用自然语言对话即可自动下载发票、建立账套，生成增值税/企业所得税/财务报表/财行税/个税五表，并逐张产出可核对的「操作卡」，辅助本人在电子税务局完成申报。读取自动、填表提交默认人工；自动报税为实验室功能，须全程盯守并逐笔确认。当前适配浙江省电子税务局小规模按季申报。触发词：开始报税/报税/下载发票/生成财报/填报台/电子税务局操作/无双报税自动版。
argument-hint: 说「开始报税」并指定公司，或「下载发票 / 生成财报」
argument-hint-en: Say 「开始报税」 and name the company, or 「下载发票 / 生成财报」
argument-hint-zh: 说「开始报税」并指定公司，或「下载发票 / 生成财报」
version: 1.2.0
user-invocable: true
---

# 无双报税自动版 · 浙江小规模纳税人季度申报自动驾驶

> 全流程按**浙江省电子税务局**界面校准；申报表行次/税额口径全国统一，他省仅需另行校准 etax 界面路径后启用。术语统一用「小规模纳税人」（勿写"小额"）。

## 本体获取与免责
- **本体**：「无双报税」是一款本地单文件 HTML 软件（同时是台账载体），下载地址 https://chuisoft.cn/wushuang-tax.html 。本 skill 与本体联动：在数据目录放置空文件 `.skill-linked` 后，本体进入联动模式（隐藏 OCR/统计/日历/实验功能，工作台显示 skill 动态卡与「切回完整模式」开关）。
- **免责**：自动报税为实验室功能，属高度敏感操作，须用户全程盯守、逐笔确认。纳税申报是纳税人法定义务，本 skill 仅提供辅助与核对，最终由本人在电子税务局提交并对申报真实性负责。坚持「一个公司一套账」，仅供本人/本公司自行申报使用，不构成涉税专业服务或代理记账。

## 分工铁律
- **读操作全自动**：下载发票、申报历史、A100000 亏损、财报模板导入、缴款回执归档。
- **写操作默认人工**：填表/提交/缴款由用户执行；skill 每张表产出「操作卡」（入口路径+要抄的数+坑+校验点）。CDP 代操仅在用户明确说"帮我点/帮我填"时启用。
- 回落纪律：税局页面改任何格→先回落本地账套/报表一致才提交；提交后先更新本地关联报表（memo/filed/归档）再做下一张。

## 0 Onboarding（首次）
1. 先问：agent 代建账套（用户给公司名/税号/类型/个税模式，可后补）or 用户自己在无双报税建（保留手动，隐私）。数据目录=用户指定的工作文件夹（data_<公司>.json，html 不存数据）。代建或首次落账后在数据目录写空文件 `.skill-linked`（进入联动模式，详见「本体获取与免责」）。
2. 基线采集（只做一次）：CDP 登录态→申报信息查询（申报日期起=年初，picker readonly 见§3）→18 行清单→A100000 详情页「导出」得官方整包→pypdf 读 A106000「可结转以后年度弥补的亏损额合计」→params.lossCarry。
3. 资产负债表年初：未分配利润=汇算逐年境内所得累加；实收资本=工商年报实缴；货币资金=银行余额反推（现余额+本期流出−流入）；差额进其他应付款=股东垫资 plug；与用户银行截图核对。

## 1 每季流程（触发：开始报税）
1. 窗口门：申报窗=季末次月 1–15 日顺延（以无双报税申报日历为准）；窗口外劝退，窗口内催办+倒计时。
2. 下票：税务数字账户→全量发票查询→⊗清空开票日期起止（或填季首）→开具+取得各查一次；属期=季 3 个月。代开/纸票不在数字账户开具库；红冲核实=红字发票业务→红字发票记录（对应蓝票号）；红蓝配对净零不入账、memo 披露。
3. 落账：sales/purchase {no,date,item,amount,tax,total,party,rate,special}；小规模 deductible=0 成本取 total；专票标 special:true（专票不论额度必缴）。
4. 报表：填报台镜像（增值税/所得税/财报/财行税/个税五卡）+核对单；财报官方导入模板=填报台按钮→报表/ 文件夹。
5. 出操作卡（顺序：财报→所得税→增值税→财行税→个税扣缴端），数字一律取自填报台点格复制。

## 2 操作卡要点（浙江 etax 实战坑录）
- 财报：导入模板一键带出；校验规则「本年累计=本期+本年上期累计」逐行比对税局系统数，分差用账套 cwbb.plx pin 已报送值；首报企业无历史财报时年初按§0.3。
- 所得税 A200000：财报提交后自动带出（营收/成本/税金/利润总额/行24弥补亏损）；利润总额小于上期会弹「核实无误，提交申报」确认框=正常（本季亏损时）。
- 增值税小规模：发票自动带出；主表列序=[货物本期,服务本期,货物年累,服务年累]；季≤30万普票免、专票不免。
- 财行税：**先税源采集后合并申报**。悬停印花税卡片→税源采集→新增税源：按期申报+属期=季；税目/子目按本公司历史申报口径（示例：技术合同类，服务类购销双方均计）；**子目必选**否则整表提交被「征收子目为空」阻断（预置零行承揽/租赁同病：录入补子目、金额 0）；计税金额=不含税合同额合计、凭证数量=发票张数；减免性质留空+合并申报表「六税两费减征=是」；有税额时**禁止一键零申报**；缴款=税费缴纳页三方协议。
- 个税：自然人电子税务局扣缴端独立线，属期次月 15 日前（顺延），etax 待办不显示。零员工且无扣缴税费种认定的公司可能无需申报——先开扣缴端看认定再定，别盲零报。**用户问个税零申报/扣缴端操作时，把 references/geshui-zero-guide.md 全文发给用户**（含人员采集→生成零工资记录→两次"未填写三险一金"弹窗继续→获取反馈判成功→导出归档 报表/申报存档/）。
- 零收入零合同公司差异：印花税可一键零申报（有合同则禁）；财报=零资产负债表（实缴/银行/未分配三数）+零利润表；残保金 0 人免征但年度零申报仍须报。
- 确认弹窗：「真实责任」四字=**4 个文本框打字**（非点选）；风险扫描框「继续申报」=提交、「修改表单」=返回。
- 黄色感叹号=警示：**先读 tooltip 再决定**（规则多为年累=本期+上年累）；红色=阻断错误。
- 延续性：表内历史季列=税局预填（历史申报），勿重写；账局分差（舍入口径）以税局数申报、账面数记账，memo 记差不更正。税局舍入口径：应纳税四舍五入→减半再四舍五入→净额=差；印花税依据不含税。

## 3 CDP 配方
- 多公司隔离：每公司独立 Chrome profile（\_etax_<公司简称>）+ 独立端口（9223 起逐公司 +1）+ 独立数据目录；etax 登录一浏览器一公司，禁止同浏览器/同窗口并发跑两家；一家做完关浏览器再换下一家。
- 专用 Chrome：port 9223、独立 profile、--remote-allow-origins=\*；不碰用户日常浏览器；同 SPA 多标签互扰→同一时刻只留一个表单标签。
- 只认真鼠标 Input.dispatchMouseEvent（坐标=getBoundingClientRect）；鼠标失灵改 JS click；SPA back() 无效→location.href 重导航+重查询。
- 日期控件 readonly：⊗清空或面板年/月下拉+点日格（dppt 域日格点击不提交→⊗清空法）；native setter 只改显示不改模型。
- 跨域 iframe 表单：Page.getFrameTree+createIsolatedWorld 后 evaluate；改值=native value setter+input/change 事件；上传=Runtime.evaluate 取 input objectId→DOM.setFileInputFiles。
- 提交链：提交申报→核实无误→继续申报→真责任打字→确定→「正在申报处理」轮→成功页；归档=详情页导出（官方件）或 printToPDF→数据目录 报表/申报存档/；缴款成功截图也归档。

## 4 核对与收尾
- 银行余额勾稽：上期余额±现金流=本期，与用户截图互验。
- 每表提交后：归档回执→memo 记实缴数→filed 标记→更新关联报表（如印花税实缴回落账套）→再下一张。
- 季结输出：四表结果+实缴合计+剩可弥补亏损+下季窗口日。

## 5 产品协同
- 本体为单文件 HTML，兼作台账载体；数据存于用户指定的工作文件夹（勿越出该目录读写）。
- 填报台镜像即操作卡数字源；本体与 skill 各自独立迭代，功能口径以本 skill 记录为准。
