# Agent 指令

本文件是给 DeepSeek Harness 在本工作区工作时的常驻约束。

## 删除操作（强制）

执行任何 shell 命令时，**删除语句必须单独执行**：

- 一条命令里只允许出现**这一个**删除语句；
- 不得用分号、&&、||、管道、换行与其它语句串联；
- 不得嵌套在代码块 {}、分组 ()、子表达式 $() 中，也不得放进
  if / foreach / for / while / switch / try；
- 不得用 -Command、-EncodedCommand、cmd /c、bash -c、Start-Process、
  Invoke-Expression / iex 包装；
- 不得在一条命令里放多个删除调用。

正确：Remove-Item -LiteralPath 'D:\tmp\a.txt' -Force

错误示例：
- Remove-Item 'a'; Write-Host done
- if ($x) { Remove-Item $x }
- ... | ForEach-Object { Remove-Item $_ }
- iex "Remove-Item a"

违反时 dsh-delete-guard 插件会在 tools/pre-execute 直接判 deny，命令不会执行。

## 删除前自检

1. 目标路径必须是**字面量**，或已单独打印确认过的变量；
2. 先确认目标不是 C:\Users\<用户> 本身、其祖先、或盘符根；
3. 变量名不要用 home / HOME / env 这类与 shell 内置冲突的名字
   （PowerShell 变量名大小写不敏感，$home 就是 $HOME）；
4. 能用"改名/移动"解决就不要"递归删除"。

## 其它

- 递归删除、批量清理时，优先逐项删除并打印每一项，不要用通配符一次匹配。
- 破坏性操作前先备份：注册表用 reg export，文件用复制。
