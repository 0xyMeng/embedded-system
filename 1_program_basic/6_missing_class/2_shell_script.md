# shell 工具和脚本



变量语法，流程控制，函数

## shell 脚本

很多情况下我们需要执行一系列的操作并使用条件或循环这样的控制流。shell 脚本的复杂性进一步提高。

shell 脚本与其他脚本语言不同之处在于，shell 脚本针对 shell 所从事的相关工作进行了优化。因此，创建命令流程（pipelines）、将结果保存到文件、从标准输入中读取输入，这些都是 shell 脚本中的原生操作，这让它比通用的脚本语言更易用。

定义变量作为学习语言的第一步。

在 bash 中为变量赋值的语法是 `foo=bar`，访问变量中存储的数值，其语法为 `$foo`。 需要注意的是，`foo = bar` 无法按照预期工作的，解释器会调用程序 `foo` 并将 `=` 和 `bar` 作为参数。在 shell 脚本中使用空格会起到分割参数的作用。

关于字符串的使用。' 里的所有内容会直接输出。" 定义的字符串里的变量会替换

m@vm:~/missing$ echo "$foo"
bar
m@vm:~/missing$ echo '$foo'
$foo

bash 也支持 if, case, while 和 for 这些控制流关键字。同样地， bash 也支持函数，它可以接受参数并基于参数进行操作。

写一个脚本名为 mcd.sh
```bash
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```
在 shell 里执行 `source mcd.sh` 看起来没什么事情发生，但是 shell 中已经有 mcd() 函数了。在终端执行 `mcd test` 就可以看到变化了。

bash 里的特殊变量
- `$0` 脚本名
- `$1` - `$9` 脚本的参数。 `$1` 是第一个参数，依此类推。
- `$@` 所有参数
- `$#` 参数个数
- `$?` 前一个命令的返回值
- `$$` 当前脚本的进程识别码
- `!!` 完整的上一条命令，包括参数。常见应用：当因为权限不足执行命令失败时，可以使用 sudo !! 再尝试一次。
- `$_` 上一条命令的最后一个参数。


命令通常使用 STDOUT 来返回输出值，使用 STDERR 来返回错误及错误码，便于脚本以更加友好的方式报告错误。 返回码或退出状态是脚本/命令之间交流执行状态的方式。返回值 0 表示正常执行，其他所有非 0 的返回值都表示有错误发生

通过 $? 来查看，退出码可以搭配 &&（与操作符）和 ||（或操作符）使用，用来进行条件判断，决定是否执行其他程序。它们都属于短路 运算符（short-circuiting） 同一行的多个命令可以用 ; 分隔。程序 true 的返回码永远是 0，false 的返回码永远是 1。几个例子

m@vm:~/missing/test$ false || echo "hello"
hello
m@vm:~/missing/test$ true || echo "hello"
m@vm:~/missing/test$ true && echo "hello"
hello
m@vm:~/missing/test$ false && echo "hello"
m@vm:~/missing/test$ false ; echo "hello"
hello
m@vm:~/missing/test$

用或运算连接两个命令，如果第一个失败了，就执行第二个。如果第一个成功了，就不执行了。

用与连接两个命令，如果第一个成功了，执行第二个，如果第一个失败了，就不执行第二个了。

此外 ; 可以分割两个命令

将一个命令的输出保存到一个变量中 `foo=$(pwd)`，通过 命令替换（command substitution）实现。两个命令嵌套。

通过 $( CMD ) 这样的方式来执行 CMD 这个命令时，它的输出结果会替换掉 $( CMD ) 。例如，如果执行 for file in $(ls) ，shell 首先将调用 ls ，然后遍历得到的这些返回值。

还有李哥小众工具 进程替换（process substitution）， `<( CMD )` 会执行 CMD 并将结果输出到一个临时文件中，并将 <( CMD ) 替换成临时文件名。这在我们希望返回值通过文件而不是 STDIN 传递时很有用。例如， `diff <(ls foo) <(ls bar)` 会显示文件夹 foo 和 bar 中文件的区别。


通过看例子来学脚本怎么写
```bash
#!/bin/bash

echo "Starting program at $(date)" # date会被替换成日期和时间

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # 如果模式没有找到，则grep退出状态为 1
    # 我们将标准输出流和标准错误流重定向到Null，因为我们并不关心这些信息
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```


## shell 工具
