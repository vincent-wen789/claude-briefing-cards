# briefing-card

[English](README.md) · **中文** · [日本語](README.ja.md)

一个 Claude Code 插件:聊天内渲染的**决策卡 / 进度卡 / 确认卡**(带可点按钮),外加 **T0–T3 汇报颗粒度路由**——每轮自动判断一次:一条状态/决策该用纯文字、快速提问、聊天内交互卡,还是 HTML 文件。

## 为什么是插件,而不只是 skill

skill 的 `description` 只对**显式触发**可靠(你开口要卡)。它扛不住**弥漫式自动触发**——「要汇报决策/进度时默认就画卡」。这种行为需要一段每轮常驻在上下文里的 standing instruction。裸 skill 文件夹不带这段,所以在新机器上「装了跟没装一样」。

本插件用一个 **SessionStart hook**(`hooks/session-start`)解决:每次会话开始 / 清屏 / 压缩时,把路由规则作为常驻上下文注入。装上即用,不用改你的 `CLAUDE.md`。

## 安装

在 Claude Code 里(不是终端 shell):

```
/plugin marketplace add vincent-wen789/claude-briefing-cards
/plugin install briefing-card@vincent-plugins
```

然后**新开一个会话**——hook 在会话开始时注入,所以装它的那个会话不受影响。

## 验证

- **快检**:新会话里问一句「你现在有没有关于 T0–T3 汇报卡的常驻指令?」它能把路由复述出来 = hook 生效。
- **真检**:别开口要卡,让它做个多线进度汇报、或给你俩带取舍的选项。它应当自动出卡/走路由,而不是甩纯文字。

## 结构

```
.claude-plugin/plugin.json        # 插件清单
.claude-plugin/marketplace.json   # marketplace 条目(source: "./")
hooks/hooks.json                  # SessionStart → 跑 session-start
hooks/session-start               # 注入常驻路由上下文的脚本
hooks/session-context.md          # 路由块(要改行为改这里)
skills/briefing-card/             # skill 本体(模板 + 路由)
```

## 说明

这是**个人版**——口吻、触发词、「默认就出卡」的倾向都是为单个用户调的,而且是中文。要发给陌生人,得先去个人化,并把激进默认改成可选(否则会过度画卡刷屏)。
