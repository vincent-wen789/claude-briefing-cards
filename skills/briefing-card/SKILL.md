---
name: briefing-card
description: Load this whenever about to report to Vincent — a decision or options to pick, a progress update across projects, or a list of items needing his sign-off — to run the T0–T3 routing and pick the right form. 汇报/进度/决策/选项/确认/拍板 默认走 T2 出卡（2026-06-18 翻转默认）——除非一句话能说清/单一事实/闲聊（降 T0 纯文字）或 2-4 选项一行讲得清（降 T1 选项卡）才降级。Reporting/进度/汇报/决策/确认 a status or asking him to choose IS the trigger — running the routing is the default, plain text is an outcome of it, not a reason to skip the skill. Also on manual triggers 画个决策卡/进度卡/确认卡 / 出个卡, and on callback messages starting with 决策卡「/进度卡「/确认卡「 (apply 接收端守则 first). Skip only for one-sentence answers, single facts, and chit-chat.
---

# briefing-card 汇报卡

> 🔴 **状态（2026-06-14 起）· 冻结开发，进入真实使用检验**
> 功能完整（三类卡 + T0–T3 路由 + 自动/手动触发 + 安全层）· 5 轮独立 review 过 · darwin 86.8。**唯一缺口是真实使用 = 0**，所有「好用 / 触发准 / 体验顺」的结论都来自 subagent 模拟。
> **除非 Vincent 撞到真实场景的具体问题（该画没画 / 卡点着别扭 / 触发漏了），否则不要再加功能、不要再 review**——边际收益已递减，复杂度利息在涨（110 行 + CLAUDE.md 两段，今天连改文件都出过「报 success 没落地」的乌龙）。下一个改动应由真实使用反馈驱动，不由「还能更完善」驱动。

## ⚠️ 装到别的机器（触发引擎不在本文件）

本 skill 的**自动触发靠 CLAUDE.md 里 `## 汇报卡 briefing-card（自动触发）` 那段**，不在 SKILL.md。只 copy 本文件夹 → 只剩手动触发（`出个卡` / 回传），自动出卡失灵（Vincent 2026-07-06 实测撞到）。装机：① copy 文件夹到目标机 `~/.claude/skills/` ② 把 `claude-md-block.md` 里 `8<` 之间的段落粘进目标机 CLAUDE.md。

## Overview

用聊天内渲染的 HTML 卡片替代纯文字，做决策请求和进度汇报。两个核心价值：**模板强制完整**（固定区块逼着填全信息）+ **决策按钮闭环**（点按钮 = 回传决策，消灭误解）。颗粒度路由是灵魂——画错时机比不画更糟。

## T0–T3 路由（先过这道闸再动手）

| 档 | 形式 | 条件 |
|----|------|------|
| T0 | 普通文字 | 一句话能说清 / 单一事实 / 闲聊。**绝不画卡** |
| T1 | AskUserQuestion | 2-4 个选项，每项利弊一行讲得清，不需要图/链接 |
| T2 | 聊天内卡片（本 skill 主体） | 决策需多维对比/证据链接/内联数据图形；进度 ≥3 条线或含 blocker；pending 确认 ≥3 项且每项带出处（≤2 项走文字或 AskUserQuestion） |
| T3 | HTML 文件 | 要存档 / 发给别人 / 要看内容图 / 超约两屏 → 转 huashu-design |

**默认翻转（2026-06-18 · Vincent 第一条真实使用反馈：手动召唤太累、触发太怂）**：一旦判定本轮属于**汇报 / 进度 / 决策 / 选项 / 确认 / 拍板**场景，**默认假设落 T2 出卡**——举证责任反过来：不再问「够不够得上 T2」，而是问「能不能安全降到 T0/T1」。只有①一句话能说清 / 单一事实 / 闲聊 → 降 T0；②2-4 个选项且每项利弊压得进一行、不需图/链接 → 降 T1。两条都不满足就出卡，别再犹豫。**翻转的是判定倾向，不是取消节流**：防卡墙节流（硬规则那条，1 小时 ≥2 张卡自动降文字）仍然全程生效，被卡片刷屏的风险由它兜底——所以放心偏向出卡。

从 T0 往上判断，停在第一个满足的档。T1 vs T2 拿不准：把每个选项压成一行（利弊 + 改动量 + 介入点）——全部压得进且不需要链接/图 → T1；任何一个选项压不进一行，或需要链接/图/对比表 → T2。注意：改动量本身不构成推力（「跨 3 文件」塞得进一行），是「装不下」才推 T2。

**T2 vs T3 边界（任一命中即 T3）**：
1. 要存档复用，或要发给 Vincent 以外的人
2. 内容里有**需要看内容的图**（截图 / 照片 / 设计稿）——无论文件多小一律 T3：聊天内卡片受 CSP 沙箱限制，外部图床和本地图片都加载不出来，且卡内小窗看不清细节。仅装饰/识别用途的小图（logo、icon，几 KB 级 base64 内嵌）可留 T2。**数据类可视化不算图**：进度条、对比条形、趋势迷你图应该用内联 SVG/CSS 直接画进 T2 卡里——「有图可看」优先靠原生图形实现，不是一律踢去 T3
3. 按模板填完预计超过约两屏、需要滚动阅读。一到两屏之间先砍信息密度或拆成两张卡留在 T2，实在装不下才升 T3

T3 注意两点：① 文件里**没有 sendPrompt 回传**——决策类内容升 T3 时，文件只承载详情（图、长对比、引用全文），决策收口仍回聊天（文件 + 一张轻量 T2 决策卡或 T1 选项卡）；② 产出走 huashu-design，给出文件路径并直接帮 Vincent 在浏览器打开。

## 前置检查（每次画卡前）

1. `mcp__visualize__show_widget` 工具可用吗？**不可用（纯终端环境）→ 输出同结构文字版**：保留区块标题（要决什么 / 选项对比 / 我的倾向…），内容用 markdown。
2. 可用 → 先静默调 `mcp__visualize__read_me`（modules 按需 mockup/interactive；platform 按宿主环境传，移动端 mobile、拿不准才 desktop），遵守其设计系统（CSS 变量、无 emoji、Tabler 图标、扁平无渐变），再调 show_widget。

## 决策卡模板（可点交互）

区块顺序固定，缺一块就是没做完：

1. **类型徽章 + 标题**：徽章固定文案「决策卡」；标题 = 要决什么一句话 + 背景一句话
2. **选项对比**：每项 = 利弊 · 改动量（1 文件 / 跨 3 文件 / 跨系统）· 需 Vincent 介入的部分 · 证据链接。耗时默认不标（2026-07-02 同步全局校准规则：实测估时普遍偏高 5 倍+）；agent 真跑 10 分钟+ 的长任务才按实际标
   - **现状锚（不决策的代价）**：选项里隐含一条「维持现状」基线——当前是什么状态、你不拍会怎样（自动按倾向走，还是僵等）。没有这条，用户算不出「不选」的成本（persona-audit 2026-06-18 ①③）
3. **我的倾向 + 理由**：必须有立场，不许各打五十大板
4. **行动按钮**：每个选项一个按钮，回传统一格式 `sendPrompt('决策卡「<标题>」(MM-DD)：<完整决策语句>')`——脱离卡片独立可读（"决策卡「X」(06-13)：选方案 A" ✓，"确认" ✗）；外加「补充说明」按钮 `sendPrompt('决策卡「<标题>」(MM-DD)：我有补充意见，先冻结执行')`；收到该回传 = 确认冻结 + 安静等输入，不追问
5. **卡外正文**：一句话复述「要决什么 + 我的倾向」，防 widget 渲染失败时信息全丢

## 进度卡模板（纯展示 · blocker 按钮是唯一受控例外）

1. **类型徽章 + 总体状态**：徽章固定文案「进度卡」；一句话总状态 + 截至时间
2. **各线状态行**：名称 + 状态点 + 一句话现状。状态点标准：绿 = 正常推进无需介入；黄 = 等 Vincent 介入或有风险；红 = 被 blocker 卡住（红点线必须同时出现在 blocker 条）。同项目非首次汇报标增量（较上次 ↑/↓/→），未变的线折叠成一行
3. **blocker 高亮条**（红底）：有 blocker 必须单列，不许埋进状态行。条内可渲染**唯一一个「处理 blocker」按钮**，仅当处理动作唯一且明确时才出现（litmus：按钮文案里写不出具体对象——哪个文件/哪条/哪个命令——就不渲染）；按钮文案 = 具体动作，回传 `sendPrompt('进度卡「<标题>」(MM-DD)：开始处理 blocker——<具体动作>')`，**blocker 按钮回传一律先复述再动工，不论耗时档**。要拍方向的 blocker 不给按钮，但**必须同回合紧跟一张决策卡或 T1 选项卡收口**（不算违反节流）——不许留红条死胡同
4. **下一步 + 链接**：每条下一步标「需你介入」标注；耗时默认不标，长任务（10 分钟+）才按实际标——与决策卡同标准
5. **卡外正文**：一句话复述总状态 + 最大 blocker，防 widget 渲染失败时信息全丢

## 确认卡模板（可点交互 · 本地勾选一次提交）

1. **类型徽章 + 标题**：徽章固定文案「确认卡」；标题 = 确认什么一句话 + 背景一句话
2. **待确认项列表**：每项 = 一句话内容 + 出处（链接/路径）+ 三态本地勾选（确认 / 驳回 / 待定），**默认按「我的建议」预填**（用户只改不同意的项），无建议才留空。勾选只改卡内 JS 状态，**严禁每项单独 sendPrompt 刷屏**；🔴 含不可逆动作的项不预填，留空强制手选
3. **我的建议**：默认推荐哪些项确认/驳回 + 一句理由，有立场
4. **提交按钮**：「提交选择」把全部选择合并成**一条**消息回传，格式 `确认卡「<标题>」(MM-DD)：确认 <项名>；驳回 <项名>；待定/未处理 <项名>`，脱离卡片可独立读懂。提交后 JS 把提交按钮置灰改「已提交」并锁定勾选——提交键永远可点会诱发双重提交。🔴 「全部确认并提交」快捷按钮**仅当「我的建议」是全部确认时才渲染**——卡上含建议驳回项时这个按钮是误触雷（一键否决建议且不可逆），禁止出现
5. **卡外正文**：一句话复述待确认项数 + 最关键一项

## packet 卡模板（handoff 融合 · save 自动 / pickup 手动）

handoff skill 的 save / pickup 两个 flow 借这套卡展示。**两张卡都不算主动汇报，不受防卡墙节流**——它们是 handoff 流程的固定步骤。无 widget（纯终端）→ 同结构文字版。

### 合体卡（handoff save · 🟢 每次 save 自动出）

packet 落盘后**自动**渲染（Vincent 2026-06-15 拍板：每次 save 自动，其余卡手动）。一张卡上下两半：

1. **类型徽章 + 标题**：徽章固定「handoff」；标题 = packet topic 一句话 + 「已存盘」
2. **上半「这趟干了啥」**：从 packet State block 提炼 3-5 条**绿勾回顾**，每条一句话 + 出处（commit / 文件路径 / 决策）。这半是回头看，已发生的事
3. **下半「接下来」**：从 packet Pending checklist 来，按优先序列待办，每条标「需你介入」；耗时默认不标（同决策卡标准）。这半是往前看，未发生的事
4. **待你拍板（🔶 条件渲染 · 仅当 packet 有悬而未决的 decision 时出）**：packet 里有还没拍板、等 Vincent 定的 open question（通常标在 State / Constraints）→ 单列一个高亮块，每条 = 问题一句话 + 倾向/选项 + 一个「确认」按钮（沿用决策卡 sendPrompt 完整语句格式，脱离卡独立可读）。**没有待拍板项就整块不渲染**（像 2026-06-15 这次 demo 就没有）。来源：Vincent「参考卡没东西要确认就别放，以后有了要能 surface」
5. **存档信息条**：slug + 绝对路径 + `pick up <slug>` 启动指令——**纯文字可复制，不做 sendPrompt 按钮**（pickup 在新 session 输入，当前 session 的按钮帮不上）
6. **「现在继续」按钮（可选）**：给"不换 session 接着干"的分支——文案 = 第一条 pending，回传 `sendPrompt('handoff「<topic>」(MM-DD)：不换 session 继续——<第一条 pending>')`。Vincent 明显要收工换 session 时不渲染
7. **卡外正文**：一句话复述——这趟干了啥 + 下一步第一条 + 存档路径（防 widget 失败信息全丢）

### packet 卡（handoff pickup · 手动 `pick up <slug>` 触发）

Vincent 输入 `pick up <slug>`、Read packet 完后渲染：

1. **类型徽章 + 标题**：徽章固定「恢复中」；标题 = slug + State 一句话
2. **Pending 列表**：checklist 形式，**首项高亮**（上次卡在这条）
3. **「从这开始」按钮**：sendPrompt 第一条未完成 pending → `pick up「<slug>」(MM-DD)：从「<第一条>」开始`
4. **决策按钮（条件渲染）**：packet 里有悬而未决的待拍板 decision（Constraints 标的）→ 加一个决策按钮收口，回传完整决策语句
5. **卡外正文**：一句话复述——State + 上次卡在哪 + 建议从哪条起

## 交互样板（2026-07-02 · 可点卡强制照抄，禁止现场发挥）

真实使用反馈：按钮时好时坏。根因 = 每张卡现场手写交互代码，写法漂移（inline onclick 引号断裂 / 写了 disabled 忘了点亮 / 流式期间 sendPrompt 未就绪）。统一修法：**payload 进 `data-send` 属性 + 卡尾唯一 `<script>` 统一绑定**。铁律：

1. **禁 inline `onclick`**——payload 含引号就断，且流式期间 sendPrompt 可能未注入。事件绑定只在卡尾 script 里做（script 保证流式完成后才执行）
2. **静态 payload 一律进 `data-send`**（HTML 属性双引号包裹，payload 内只用中文标点、禁英文双引号）；按钮初始写 `disabled`，label 末尾带 ` ↗`
3. **确认卡三态勾选/提交不走 data-send**——payload 按勾选状态动态拼，写法照抄 B 段
4. 各模板节里写的 `sendPrompt('…')` 只定义**回传文案格式**，不是字面 HTML 写法——落地一律按本节样板
5. **选中/状态类视觉一律 JS 写 inline style**——`<style>` 块 + CSS 属性选择器在宿主里不生效（2026-07-02 实测：预填高亮全白、点击不变色，JS 逻辑却照常跑），排版样式也一律 inline `style="…"`
6. **禁止程序化 `.click()` 转发触发 sendPrompt**——isTrusted（是否真实用户手势）= false 时宿主可能拒发。多个按钮共用提交逻辑时抽成函数直接调，保持真实点击调用栈

### A · 静态回传按钮（决策卡 / blocker 按钮 / packet 卡）

```html
<button class="bc-act" data-send="决策卡「标题」(MM-DD)：选方案 A——完整决策语句" disabled>选 A · 短标签 ↗</button>

<script>
(function(){
  var btns = document.querySelectorAll('.bc-act');
  if (!btns.length) return;
  if (typeof sendPrompt !== 'function') {
    btns.forEach(function(b){ b.style.display = 'none'; });
    var tip = document.createElement('p');
    tip.style.cssText = 'font-size:13px;color:var(--text-muted);margin:8px 0 0';
    tip.textContent = '这里按钮点不了，直接打字回复就行。';
    btns[btns.length - 1].insertAdjacentElement('afterend', tip);
    return;
  }
  btns.forEach(function(b){
    b.disabled = false;
    b.addEventListener('click', function(){ sendPrompt(b.dataset.send); });
  });
})();
</script>
```

### B · 确认卡三态勾选 + 一次提交

要点：预填直接在 HTML 给按钮写 `data-on="1"`；不可逆项三态全不带（留空强制手选）；没预填也没手选的项提交时自动归「待定/未处理」，不会漏项；「全部确认并提交」（`id="bc-all"`）仅当建议 = 全部确认时才写进 HTML（硬规则不变），与提交键共用 `doSubmit()`。三态按钮不带 ↗（不发消息），提交/bc-all 带 ↗。提交后锁定文案「已提交 · 勾选已锁定」+ 隐藏 bc-all——锁定态必须肉眼可见，否则锁定后点三态没反应会被当成坏了（2026-07-02 第一张测试卡真实翻车点，连同 style 块失效、`.click()` 转发拒发共三处，铁律 5/6 由此而来）。

```html
<div class="bc-item" data-name="项名A">…一句话内容 + 出处…
  <span class="bc-tri">
    <button data-val="确认" data-on="1" disabled>确认</button>
    <button data-val="驳回" disabled>驳回</button>
    <button data-val="待定" disabled>待定</button>
  </span>
</div>
<button id="bc-all" disabled>全部确认并提交 ↗</button>
<button id="bc-submit" data-prefix="确认卡「标题」(MM-DD)" disabled>提交选择 ↗</button>

<script>
(function(){
  var items = document.querySelectorAll('.bc-item');
  var submit = document.getElementById('bc-submit');
  var all = document.getElementById('bc-all');
  if (!submit) return;
  if (typeof sendPrompt !== 'function') {
    document.querySelectorAll('.bc-tri, #bc-submit, #bc-all').forEach(function(el){ el.style.display = 'none'; });
    var tip = document.createElement('p');
    tip.style.cssText = 'font-size:13px;color:var(--text-muted);margin:8px 0 0';
    tip.textContent = '这里按钮点不了，直接打字回复：确认哪几项、驳回哪几项。';
    submit.insertAdjacentElement('afterend', tip);
    return;
  }
  var C = {
    '确认': 'background:var(--bg-success);color:var(--text-success);border-color:var(--border-success)',
    '驳回': 'background:var(--bg-danger);color:var(--text-danger);border-color:var(--border-danger)',
    '待定': 'background:var(--surface-1);border-color:var(--border-stronger)'
  };
  function paint(tri){
    tri.querySelectorAll('button').forEach(function(x){
      var on = x.dataset.on === '1';
      x.style.cssText = on ? C[x.dataset.val] : '';
      x.setAttribute('aria-pressed', String(on));
    });
  }
  var locked = false;
  function doSubmit(){
    if (locked) return;
    locked = true;
    var g = { '确认': [], '驳回': [], '待定/未处理': [] };
    items.forEach(function(it){
      var on = it.querySelector('.bc-tri button[data-on="1"]');
      var k = on ? (on.dataset.val === '待定' ? '待定/未处理' : on.dataset.val) : '待定/未处理';
      g[k].push(it.dataset.name);
    });
    var parts = [];
    Object.keys(g).forEach(function(k){ if (g[k].length) parts.push(k + ' ' + g[k].join('、')); });
    sendPrompt(submit.dataset.prefix + '：' + parts.join('；'));
    submit.disabled = true;
    submit.textContent = '已提交 · 勾选已锁定';
    if (all) all.style.display = 'none';
    document.querySelectorAll('.bc-tri button').forEach(function(b){ b.disabled = true; });
  }
  document.querySelectorAll('.bc-tri').forEach(function(tri){
    paint(tri);
    tri.querySelectorAll('button').forEach(function(b){
      b.disabled = false;
      b.addEventListener('click', function(){
        tri.querySelectorAll('button').forEach(function(x){ delete x.dataset.on; });
        b.dataset.on = '1';
        paint(tri);
      });
    });
  });
  submit.disabled = false;
  submit.addEventListener('click', doSubmit);
  if (all) {
    all.disabled = false;
    all.addEventListener('click', function(){
      document.querySelectorAll('.bc-tri button').forEach(function(x){
        if (x.dataset.val === '确认') { x.dataset.on = '1'; } else { delete x.dataset.on; }
      });
      document.querySelectorAll('.bc-tri').forEach(function(tri){ paint(tri); });
      doSubmit();
    });
  }
})();
</script>
```

## 硬规则

- 每个结论/状态带出处（链接或文件路径）；给不出就标「未验证」。链接旁附 ≤6 字关键摘录，不点开也能扫到要点
- 防卡墙节流（跨类型计数）：同 session 约 1 小时内已出 ≥2 张卡 → 第 3 张起降文字；同项目 1 小时内再报进度只发文字增量；session 内 ≥3 张未回传决策卡 → 新决策走文字
- 所有 sendPrompt 回传文案统一前缀 `<卡类>「<标题>」(MM-DD)：`——日期戳是跨 session 识别旧卡的唯一线索
- **按钮可见文字 ≠ 回传 payload**（persona-audit 2026-06-18：①②③ 三镜头撞上，主用户判这是「会导致退回打字、弃用卡片」的最脆点）：按钮**脸上显示**的字要短、人话、自解释——`选 A · GLM 5.2`、`修这个 blocker`；`决策卡「X」(MM-DD)：…` 那串完整回传语句和技术方案全文（`从 a.download 换成 FileReader`）只进 sendPrompt 的**参数**，别显示在按钮 label 上。回传体是给系统识别旧卡用的，漏到用户眼前像机器自言自语，直接违反「说人话」
- 三类卡所有交互控件初始 disabled 置灰、script 加载完点亮——流式渲染期间点了没反应会被当成坏了。实现必须照抄「交互样板」节，不许每张卡自创写法
- 移动端：「一屏」按当前设备算；决策卡选项纵向堆叠；确认卡每项两行堆叠、控件触达 ≥44px，项数 ≥5 拆卡或升 T3
- 卡内文案遵守说人话规则：英文术语首次出现带中文括号解释、利弊/现状用白话一行讲清、讲效果不讲原理。**实现级术语**（`a.download`/`FileReader`/`IndexedDB`/`episode-collapse` 这类）即使对 Vincent 也换成效果话（「Safari 导出点了没反应、得换个写法」）；`ship`/`blocker`/`模型` 这种他熟的保留不翻——边界是「他嫌不嫌」，不是「是不是英文」（persona-audit 2026-06-18 ①②③）
- 单卡一屏内；无动画无装饰；信息密度优先
- 解释性文字写在卡外的回复正文；卡内只放结构化信息
- 示意/猜测的数据必须标「示意」
- **不可逆/花钱/合规相关的决策项**：给事实 + 利弊让 Vincent 自己定，不预填、不给「我建议选 X」的强推荐——涉及钱/法务时我不是顾问，只供他做决定所需的信息（把确认卡不预填规则升成跨卡通则；其余决策仍守决策卡「必须有立场」）
- 卡画完 ≠ 任务完成：决策卡画完要等回传，不许把"卡已发出"当作"已决策"继续往下做

## 接收端守则（收到卡片回传消息后）

- 🔴 回传选项对应真长任务（10 分钟+）或含不可逆动作 → 先一句话复述将要做什么再开工，给 Vincent 留打断窗口
- 回传内容与已决/已完成事项重复 → 大概率是翻历史误点了旧卡（旧卡按钮永远活着），先确认再执行，不许顺从地重做一遍
- 🔴 回传日期戳早于今天 → 按旧卡处理：先 mem-search 当时的 context，跟 Vincent 确认后再执行
- 同一张卡收到多条矛盾回传 → 以最后一条为准，一句话复述差异确认
- 新 session 开工时记忆系统能检索到未回传的卡 → 文字列出未决项，重新收口

## 降级与 failsafe

- Vincent 说「别画了」「文字说」→ **本 session 所有类型卡全部降级文字版**（默认全降；仅当他点名某一类如「进度别画卡」才按类降）。触发全降时回一句 ≤10 字确认（「已全降，可点名只降某类」），留低成本反悔口
- 降级期间 Vincent 用任何手动召唤词（做个 briefing / brief 一下 / 给我个概览 / 画个X卡…）→ 覆盖降级（最新指令优先），降级仅对自动触发持续生效
- 降级后的复杂决策：详情走正文文字，收口优先用 AskUserQuestion 选项卡，保住点选闭环
- 跨 session 同类场景累计被拒 ≥3 次（以记忆系统能检索到的拒绝记录为准，检索不到就不提议）→ 提议存 `feedback_briefing_card_skip_<场景>.md`（与既有 feedback_* 文件同目录），此后该场景永久走文字。每次被拒当场记一条 observation（场景+日期）——不写入，≥3 次永远凑不齐

## 已落地记录

- ✅ **handoff 融合（2026-06-14 标记 / 2026-06-15 真落地）**：save 出「合体卡」(🟢自动) · pickup 出「packet 卡」(手动)，模板见上「packet 卡模板」节。⚠️ **2026-06-14 标了"已落地"但模板节根本没建**（两处 dangling reference 指向不存在的节，handoff save flow 仍只吐文字）——正是本 skill line 10 警告的"报 success 没落地"乌龙。2026-06-15 补全两个模板 + 接通 handoff save/pickup flow 才真正落地。教训：标 ✅ 前确认文件里真有那一节。卡片借 handoff 的触发词（pick up / 存 handoff）稳定出现，绕开 briefing-card 自身语义触发弱的根因——这是当前最可靠的实际触发路径。

## Common mistakes

| 错误 | 纠正 |
|------|------|
| 每条回复都画卡 | 路由表是闸门，T0/T1 占日常大多数 |
| 按钮文案是「确认」「OK」/ 把完整 payload 写在按钮脸上 | 按钮 label 短人话（「选 A · X」）；完整决策语句只进 sendPrompt 的参数，不显示在按钮上 |
| 跳过 read_me 直接 show_widget | 必先 read_me，否则样式断裂 |
| 进度卡塞按钮 | 唯一例外是 blocker 条内「处理 blocker」按钮（动作唯一明确时）；其余一律纯展示，要决策拆决策卡 |
| 确认卡逐项点击逐条发消息 | 勾选攒在卡内 JS 状态里，「提交」一次合并回传 |
| 把长解释塞进卡里 | 卡内结构化信息，解释放正文 |
| 现场手写 inline onclick / 每张卡自创交互代码 | 照抄「交互样板」节——payload 进 data-send、卡尾统一 script 绑定。写法漂移 = 按钮时好时坏的根因（2026-07-02 真实反馈） |
