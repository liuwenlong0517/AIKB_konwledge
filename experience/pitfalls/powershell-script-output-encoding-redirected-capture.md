---
id: aikb:experience:pitfalls:powershell-script-output-encoding-redirected-capture
type: pitfall
status: verified
tags: [windows, powershell, encoding, utf-8, gbk, claude-code, git-bash, maintenance-scripts]
applicable_versions: "Windows 11；PowerShell 7 与 Windows PowerShell 5.1；Claude Code Bash 工具（Git Bash 捕获）；验证于 2026-08-27"
last_verified: 2026-08-27
review_when: "PowerShell 改变原生命令输出编码默认行为，或 Claude Code、CI 等捕获端改变输出解码方式时"
supersedes: []
relations:
  - type: related_to
    target: aikb:experience:pitfalls:windows-agent-hook-shell-encoding-boundaries
---

# PowerShell 脚本中文输出被按 UTF-8 解码成乱码：须在脚本顶部固定输出编码

## 触发条件

中文 Windows（默认代码页 GBK）上，任何用 `Write-Host`/`Write-Output` 或 `ConvertTo-Json` 输出中文的 PowerShell 脚本，在同时满足以下情况时会产生乱码：

- 脚本输出被"按 UTF-8 解码"的消费者捕获，例如 Claude Code 的 Bash 工具（Git Bash harness）、CI 日志、Git Bash 重定向；
- 脚本没有在**最顶部、首个输出之前**固定 `[Console]::OutputEncoding` 与 `$OutputEncoding`。

真实 Windows Terminal / PowerShell 控制台不会复现（PowerShell 7 控制台默认 UTF-8），因此"自己在终端里跑正常"不能排除该问题。

## 风险

- 中文变成 `�ṹУ��ͨ����` 之类的替换字符与乱码，容易误判为数据损坏或编码库问题。
- **修复点放错位置会无效**：PowerShell 管道的输出编码在首次写入时绑定，脚本中途设置 `[Console]::OutputEncoding` 不影响后续输出；固定必须放在脚本顶部、首个输出之前。
- 与既有的 hook 编码边界（[[windows-agent-hook-shell-encoding-boundaries]]）同属 Windows 编码边界家族，但触发点不同：hook 陷阱在机器对机器的 stdio JSON 数据链路，本陷阱在维护脚本的人类可见输出。排查时不要混为一谈。

## 规避方式

在脚本 `param()` 块之后、**任何输出之前**，与 hook 包装器保持一致地固定三行：

```powershell
[Console]::OutputEncoding = [Text.UTF8Encoding]::new($false)
[Console]::InputEncoding  = [Text.UTF8Encoding]::new($false)
$OutputEncoding = [Text.UTF8Encoding]::new($false)
```

- 必须放在首个输出（首个 `Write-Host`/`Write-Output`/管道输出）之前，否则不生效。
- 输出型脚本（`Write-Host` 中文）与 JSON 型脚本（`ConvertTo-Json`）都需要；后者输出路径、原因等中文字段同样受影响。
- 字节判别：UTF-8 编码的"结"首字节是 `E7 BB`，GBK 是 `BD E1`；用 Python `subprocess.run([...], capture_output=True)` 抓原始 stdout 判断，不要只看终端显示。

## 验证

- 用 Python 抓取脚本原始 stdout，按 UTF-8 解码应无替换字符、首字节为 `E7 BB` 而非 `BD E1`。
- AIKB 于 2026-08-27 对 8 个维护脚本（`install-all`、`install-root-instructions`、`uninstall-all`、`validate-performance`、`validate-setup`、`validate-structure`、`set-aikb-home`、`setup-aikb`）修复并验证：`validate-structure.ps1` 输出首字节 `e7 bb 93`，`validate-setup.ps1` 端到端输出全为干净中文，8 个脚本 Parser 语法检查通过。
- 回归需同时覆盖两种环境：真实 Windows Terminal（正常，PS7 默认 UTF-8）与重定向捕获（修复后应正常）。

## 适用范围

- 适用于 Windows 上 PowerShell 脚本输出被非控制台 UTF-8 消费者捕获的场景。
- 不适用于 hook、MCP、检查点等机器对机器数据链路——它们已通过 Python `sys.stdout.reconfigure(encoding="utf-8")` 与 wrapper 固定编码，字节正确（见 [[windows-agent-hook-shell-encoding-boundaries]]）。
- macOS/Linux 非 GBK 默认、非 PowerShell 环境不适用本陷阱。

## 关联信息

- 控制仓 `system/adapters/shared/aikb-hook.ps1`：hook 数据链路顶部固定编码的参考实现。
- 控制仓 8 个维护脚本顶部：与 wrapper 一致的固定三行。
- 相关条目：[[windows-agent-hook-shell-encoding-boundaries]]。
