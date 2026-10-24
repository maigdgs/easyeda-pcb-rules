# 嘉立创 PCB 规则助手 · easyeda-pcb-rules

一个用中文帮助新手检查和设置 PCB 规则约束，并整理嘉立创原理图绘图规范的 Codex Skill。

它能把线宽、间距、过孔、网络分类和 DRC 结果放在一起分析，解释问题、指出缺失参数，并在用户明确要求时配置规则、回读验证。没有通用固定规则包；电压、电流、铜厚和板厂工艺不同，合适的约束也会不同。

## 先理解两部分

- **这个 Skill 装在 Codex 中**：告诉 AI 如何检查和设置 PCB 规则，以及如何按要求整理原理图。
- **Run API Gateway 装在嘉立创 EDA 专业版中**：让 Codex 通过 `easyeda-api` Skill 和本地 Bridge 连接 EDA。

分析截图或导出的规则时可以离线使用。要读取或操作正在打开的原理图或 PCB，需要上面的实时连接正常。

## 安装

在 Codex 中发送：

```text
使用 skill-installer，从 https://github.com/maigdgs/easyeda-pcb-rules 安装根目录的 Skill，名称为 easyeda-pcb-rules。
```

也可以在 GitHub 点击 **Code → Download ZIP**，解压后将包含 `SKILL.md` 的整个文件夹放入 Codex 的用户技能目录，文件夹命名为 `easyeda-pcb-rules`。

Windows 默认位置：

```text
%USERPROFILE%\.codex\skills\easyeda-pcb-rules\SKILL.md
```

确认 `SKILL.md` 直接在技能文件夹内，避免多套一层目录。安装后从下一条消息开始使用；若未识别，再重新打开 Codex。

实时操作还需安装嘉立创官方 [easyeda-api Skill](https://github.com/easyeda/easyeda-api-skill) 和 [Run API Gateway](https://github.com/easyeda/eext-run-api-gateway)，在 EDA 中启用扩展及“允许外部交互”，再确认 Bridge 已连接。

## 第一次这样问

```text
使用 $easyeda-pcb-rules，只读检查我指定工程的 PCB 规则。
先告诉我现在有哪些规则，以及还缺哪些设计参数，不要改图。
```

还可以说：

```text
使用 $easyeda-pcb-rules，为我的电机驱动 PCB 制定约束建议。
请先询问会影响线宽、间距和过孔选型的必要参数。
```

```text
使用 $easyeda-pcb-rules，解释当前 PCB 的 DRC 报错，先处理最影响连通性的问题。
```

设置规则时，请写清工程/PCB、适用网络和目标值。Skill 会保存本地原值、执行请求范围内的设置并回读检查；无法确定接口或参数时会说明原因并给出手动步骤。

## 原理图绘图要求

创建或整理原理图时，Skill 强制采用以下约定：

- 图内不放 `INPUT FILTER`、`REGULATOR` 等功能分区标题，也不放工作条件、计算或调试等长篇工程说明；这些内容放在图外文档中。
- 导线只使用水平和垂直直线段，以 90° 转折，不画斜线或曲线。电感线圈、二极管等标准元件符号本体保持正常画法。
- 所有 GND 引脚向下汇入同一条底部水平 GND 总线，总线下方中央只放一个实际连接的 GND 符号。
- 保留元件与引脚身份、网络和连接关系，完成后比较网表并运行 ERC/原理图 DRC。

例如：

```text
使用 $easyeda-pcb-rules，按原理图绘图要求整理我指定的图页。
移除功能分区标题和长篇说明，统一正交连线与底部共地，保持电路连接不变并检查 ERC。
```

## 包含什么

```text
SKILL.md                              技能入口与工作方法
agents/openai.yaml                    Codex 界面信息
references/constraint-checklist.md     参数采集与规则检查清单
references/easyeda-rules.md            嘉立创规则接口、单位与版本限制
```

技能文件通过结构校验，并用模拟场景审查行为。没有以修改真实 PCB 的方式测试所有客户端版本；部分嘉立创规则 API 是 BETA，实际使用必须核对当前版本能力。DRC 通过不能替代载流、温升、EMC、绝缘或通电测试。

仓库只分发技能与通用说明。提交反馈时，请先移除电路、客户和账号等非公开信息。

## 许可与关系

本项目使用 MIT 许可，供学习、修改和分享。这是独立制作的 Skill，与嘉立创或 OpenAI 没有官方隶属关系；相关产品和 API 的权利归各自所有者。
