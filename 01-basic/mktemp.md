# mktemp

```text
mktemp 是 Linux/Unix 系统中非常常见的标准命令.

它的作用是：安全地创建临时文件或临时目录，并返回其路径。
```

1. 创建临时文件
   mktemp
   输出类似：/tmp/tmp.XXXXXXXXXX


2. 创建临时目录
   mktemp -d
   输出类似：/tmp/tmp.abc123

3. 指定文件名模板
   mktemp /tmp/myapp.XXXXXX
   其中 XXXXXX 会被替换为随机字符，避免文件名冲突。

4. 在 Shell 脚本中使用
```shell
# 创建临时文件
TEMP_FILE=$(mktemp)
# 写入内容
echo "hello world" > "$TEMP_FILE"
# 使用临时文件
cat "$TEMP_FILE"
# 删除临时文件
rm -f "$TEMP_FILE"
```

## mktemp命令的输出是什么? 为什么可以赋值给一个变量?

核心理解： 
$(...) 并不是获取命令的返回值（退出码），而是获取命令的标准输出。命令退出码需要通过 $? 获取。

```text
mktemp 命令的输出是：新创建的临时文件的完整路径。

TEMP_FILE=$(mktemp)

1. 为什么可以赋值给变量？
这里涉及 Shell 的一个语法特性：命令替换（Command Substitution）。

$(命令)
它的执行过程是：
1. Shell 执行括号中的命令。
2. 捕获命令的标准输出（stdout）。
3. 将标准输出的内容替换到 $(...) 所在的位置。
4. 再执行外层表达式。
```

## mktemp 经常与trap一起使用

mktemp 经常与 Shell 的 trap 命令配合使用，主要目的是：创建临时文件或目录，并在 Shell 脚本退出时自动清理，避免遗留临时文件。

1. 一个典型示例
```shell
#!/bin/bash

# 创建临时文件
TEMP_FILE=$(mktemp)

# 注册清理动作
trap 'rm -f "$TEMP_FILE"' EXIT

# 使用临时文件
echo "hello world" > "$TEMP_FILE"

cat "$TEMP_FILE"
```

这里涉及两个命令：
- mktemp：创建临时文件，并输出文件路径。
- trap：注册信号或 Shell 特定事件的处理动作。


2. 理解 trap
```text
trap 的基本语法：
trap '执行的命令' 触发条件

例如：
trap 'echo "脚本退出了"' EXIT
其中：
- EXIT：Shell 退出时触发。
- INT：收到 SIGINT 信号时触发，通常对应 Ctrl+C。
- TERM：收到 SIGTERM 信号时触发。

因此：
trap 'rm -f "$TEMP_FILE"' EXIT
意味着无论脚本正常执行结束，还是执行过程中发生错误导致 Shell 退出，都会执行清理命令（前提是 Shell 正常执行退出处理）。
```

```text
需要注意：trap EXIT 不能保证在 SIGKILL（kill -9）、断电或进程崩溃等情况下执行。因此，它不是绝对可靠的临时文件清理机制。

严格来说，不存在能够保证在所有异常情况下都执行清理的 Shell 机制.
实际工程中，通常采用多层防护，而不是寻找一个绝对可靠的清理命令。
1. 第一层：trap EXIT，处理正常退出
2. 第二层：临时目录使用唯一名称
    这样即使上一次执行异常退出，下一次执行也不会覆盖之前的临时目录。
3. 第三层：定期清理过期临时文件
    这是解决异常退出后遗留文件的关键。
    
对于普通 Linux Shell 脚本，采用 mktemp -d + trap EXIT + 定期清理 就是比较成熟、实用的方案。
如果临时文件承载重要业务数据，则不应该简单地依赖临时文件和 trap 管理，而应该考虑：
- 明确的资源生命周期。
- 原子写入与提交。
- 异常恢复机制。
- 定期扫描与垃圾回收。        
```