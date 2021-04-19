文档: https://golang.org/pkg/

## os

os 包提供了独立的平台接口去操作系统功能

### exec

`exec` 运行外部命令. 它封装了 `os.StartProcess` 使得在 stdin, stdout, 连接 I/O 和 管道, 和做其他调整的交互更容易.	

不像 "system" 库调用 C 和其他语言, `os/exec` 包故意(intentionally) 不调用系统 shell, 不扩展通配符或支持其他扩展, 管道, 或重定向.

