# Destructive Change Guard

`destructive-change-guard` 是一个 Codex skill，用来在删除、覆盖、重写、回滚、重命名、移动等破坏性操作前加安全检查。

它防的不是“不能删东西”，而是防止 AI 一不留神把用户已有信息干没了。这个锅太重，谁背谁秃。

## 它保护什么

这个 skill 把保护对象定义为“信息”，不只是文件本身。

它会保护：

- 代码逻辑
- 注释和设计理由
- 文档和提示词
- 配置含义
- 测试和示例
- 用户写过的文字
- 数据和记录
- imports、链接、路由、资源、引用关系
- 仓库状态和未提交改动

## 什么时候使用

只要任务可能让已有信息消失，或者让信息变得难以恢复，就应该使用这个 skill。

典型场景：

- 删除文件、目录、代码、文档、测试、配置或数据
- 范围不清楚的 cleanup 请求
- 明明可以局部修改，却准备重写整个文件
- 批量搜索替换
- `git reset`、`checkout`、`clean`、rollback、discard
- 数据库 `DELETE`、`DROP`、`TRUNCATE`
- 可能破坏引用关系的重命名或移动
- 同一任务里反复做多个“小删除”，累计后变成大风险
- 普通编辑时，作为副作用静默删掉注释、测试、逻辑、引用或用户文字

## 核心行为

- 对低风险小改走轻量流程。
- 对删一个空行、修一个明显错字这类无害操作提供快速豁免。
- 对中高风险操作要求更完整的检查。
- 检查目标、操作类型、归属、范围、信息损失和可恢复性。
- 跟踪同一任务里的累计破坏性操作。
- 把 `partial` 部分恢复当成真实风险处理。
- 明确重命名和移动也可能是破坏性操作。
- 不把“有 Git”当成万能安全网。
- 高风险操作必须获得明确用户确认。

## 安全模型

中高风险破坏性修改前，需要判断：

- 修改目标
- 操作类型
- 信息损失风险
- 恢复等级
- 内容归属
- 影响范围
- 是否考虑过低损替代方案
- 下一步动作

中高风险操作还需要 Recovery Gate：

- 是否在 Git 仓库内
- 是否有可恢复基线
- 恢复是否完整
- 工作区是否干净
- 是否存在用户未提交改动
- 备份方式
- 远程备份状态

## 禁止自动执行的命令

这些命令不能由 agent 自动执行：

```text
rm -rf
del /f /q
rmdir /s /q
git reset --hard
git checkout -- <path>
git clean -fd
git clean -fdx
truncate
drop table
delete from <table>
```

这些操作必须有明确用户意图和确认。

## 文件结构

```text
destructive-change-guard/
├── SKILL.md
└── references/
    ├── change-check-template.md
    ├── never-auto-execute.md
    ├── recovery-gate.md
    ├── risk-levels.md
    └── safer-alternatives.md
```

## 安装

可以直接 clone 或复制这个目录到你的 Codex skills 目录。

示例：

```powershell
git clone https://github.com/shawns-yao/destructive-change-guard.git
```

然后把 skill 文件夹放到你的 Codex 本地 skills 搜索路径里。

## 说明

这个 skill 会偏保守，但不是无脑保守。它包含快速豁免规则，用来避免“删一个空行也要开安全大会”的尴尬。

它的目标不是禁止删除，而是避免误删、误覆盖、误回滚，尤其是那些恢复困难、归属不明、影响范围不清楚的改动。
