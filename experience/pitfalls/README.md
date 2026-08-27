# 工程陷阱

本目录存放已经验证的工程陷阱、触发条件和规避方式。不得记录仅凭猜测得出的风险结论。

## 条目索引

- [Windows Agent Hook 必须分别固定 Shell 解释器与 UTF-8 进程边界](windows-agent-hook-shell-encoding-boundaries.md)：区分 PowerShell 命令被 Git Bash 误解析与 Python/PowerShell 标准流编码冲突，并给出真实 handler 的诊断、修复和回归方式。
- [PowerShell 脚本中文输出被按 UTF-8 解码成乱码：须在脚本顶部固定输出编码](powershell-script-output-encoding-redirected-capture.md)：维护脚本在重定向/非控制台环境默认输出 GBK，被按 UTF-8 解码的消费者捕获时乱码；固定编码必须放在脚本顶部首个输出之前。
