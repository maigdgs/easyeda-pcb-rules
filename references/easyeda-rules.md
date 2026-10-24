# 嘉立创规则接口与限制

本文件是 API 使用索引，不是稳定的规则 JSON schema。接口可能为 BETA；操作前通过 `easyeda-api` Skill 的本地参考或嘉立创当前官方文档核对完整签名、备注和版本，并检查实际可用性。

## 目标和单位

- 使用 `dmt_Project.getCurrentProjectInfo()`、`dmt_SelectControl.getCurrentDocumentInfo()` 识别工程和文档；PCB 操作要求对应 PCB 文档处于活动状态。
- Bridge 的 `/eda-windows` 列出连接窗口；`POST /execute` 的 `windowId` 指向明确窗口。不要把一个窗口的 PCB UUID 用于另一个窗口。
- [SYS_Unit](https://prodocs.lceda.cn/cn/api/reference/pro-api.sys_unit.html) 说明 PCB 数据单位为 mil，显示单位可独立设置。几何换算：`mm / 0.0254 = mil`。例如 `0.20 mm` 约为 `7.874 mil`；这只是单位示例，不是推荐间距。
- 当前规则返回的不透明字段需要逐字段核实单位，不能一概当成 mil。

## 读取规则

以下方法属于 `eda.pcb_Drc`，应分别检查当前版本是否具备：

```typescript
getCurrentRuleConfiguration(): Promise<Record<string, any> | undefined>
getCurrentRuleConfigurationName(): Promise<string | undefined>
getAllRuleConfigurations(includeSystem?: boolean): Promise<Array<Record<string, any>>>
getRuleConfiguration(configurationName: string): Promise<Record<string, any> | undefined>
getAllNetClasses(): Promise<Array<IPCB_NetClassItem>>
getNetRules(): Promise<Array<Record<string, any>>>
getNetByNetRules(): Promise<Record<string, any>>
getRegionRules(): Promise<Array<Record<string, any>>>
```

网络名称可从 `eda.pcb_Net.getAllNetsName()` 读取；`IPCB_NetClassItem` 描述 `name`、`nets` 和 `color`。类成员、规则定义和规则分配要分别核实。

官方入口：[PCB_Drc](https://prodocs.lceda.cn/cn/api/reference/pro-api.pcb_drc.html)、[PCB_Net](https://prodocs.lceda.cn/cn/api/reference/pro-api.pcb_net.html)。本地参考在 `easyeda-api/references/classes/PCB_Drc.md`、`PCB_Net.md` 和 `references/interfaces/IPCB_NetClassItem.md`。

## 写入规则

`eda.pcb_Drc` 提供以下整包覆写接口：

```typescript
overwriteCurrentRuleConfiguration(ruleConfiguration: Record<string, any>): Promise<boolean>
overwriteNetRules(netRules: Array<Record<string, any>>): Promise<boolean>
overwriteNetByNetRules(netByNetRules: Record<string, any>): Promise<boolean>
overwriteRegionRules(regionRules: Array<Record<string, any>>): Promise<boolean>
```

公开类型未定义内部稳定字段。不要自行发明 `minWidth`、`clearance`、`viaSize` 等键。官方示例中 `getCurrentRuleConfiguration()` 返回包装对象，写回的是其中 `current.config`；实际调用仍需确认当前版本的真实结构。

正确过程是读取完整原值、保存本地快照、在已核实字段上最小修改、保留其余字段、在写前再次核实目标 PCB 和原值未变、覆写对应完整配置、回读比较，并检查规则适用范围。目标或原值已变时重新读取并构造修改。`false`、异常、超时均不能视为成功；超时后先回读再重试。未能核实字段结构时给出界面设置步骤。

`createNetClass()`、`addNetToNetClass()` 和 `removeNetFromNetClass()` 也属于 `pcb_Drc`。在创建类或修改成员前核对完整签名及颜色参数；创建类不会自动产生该类的电气/几何约束。

修改当前 PCB 规则不会自动更改已有图元尺寸。`pcb_PrimitiveLine` 的线宽和 `pcb_PrimitiveVia` 的尺寸操作改变的是几何。修改板层或设置新建 PCB 的全局默认值，另属不同作用范围。

官方说明：[设计规则](https://prodocs.lceda.cn/cn/pcb/design-design-rule/)、[网络类管理器](https://prodocs.lceda.cn/cn/pcb/design-net-class-manager/index.html)、[约束区域](https://prodocs.lceda.cn/cn/pcb/place-constraint-region/)、[覆写当前规则](https://prodocs.lceda.cn/cn/api/reference/pro-api.pcb_drc.overwritecurrentruleconfiguration.html)。约束区域说明给出的覆盖顺序为区域规则 > 网络间规则 > 网络规则；针对实际版本及对象再核实。

## DRC 的返回含义

先阅读 [check 的完整签名](https://prodocs.lceda.cn/cn/api/reference/pro-api.pcb_drc.check.html)。在已确认 PCB 文档后：

```javascript
return await eda.pcb_Drc.check(true, false, false);
```

上例返回布尔值，表示本次检查是否通过。若需要详细结果，第三个参数为 `true`：

```javascript
return await eda.pcb_Drc.check(true, false, true);
```

详细结果是分组树。其顶层长度不能当作违规总数。需要计数时先识别返回结构中的叶子违规项；不能可靠识别时原样保留分组，并报告无法确定总数。记录所用参数、启用范围和前后状态；存在“通过”或“不通过”的明确布尔结果时，优先据其表述通过状态。

## 版本能力和降级

先用 `sys_Environment.getEditorCurrentVersion()` 获取版本，再以 `typeof` 等方式检查将使用的方法。文档中实时 DRC 控制接口、部分物理叠层接口与 `pcb_Layer.getCurrentLayer()` 标注 v4.2 新增，不能假定 v4.1 已具备。

接口不支持时保留可完成的读取或分析工作，明确缺少哪一项，并给出用户在对应版本界面完成该项的步骤。不要为了取得“支持”而修改用户板层、工程格式或升级软件。

只读采集不要照搬官方示例里先创建测试对象、再删除测试对象的演示部分；直接调用查询方法即可。
