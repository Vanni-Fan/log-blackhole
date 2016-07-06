# log-blackhole
日志黑洞是指，所有往这里面写的数据会直接丢弃。

在开发或调试的时候，往往会打印很多日志，这些日志大多数情况下只是为了方便开发中而设计，当你的程序被放在后端执行时，再想要获取标准输出变得很麻烦（目前可用的方法，可能就是 cat /proc/$pid/fd/* 了，但不一定有效），一般请求是把输出生成到某一个文件，然后避免太大又开始分文件保存，然后……

假设有一种方案，程序的日志输出，不占用任何磁盘空间，并且可以在后端运行，想看日志随时可看，不看的时候就直接丢弃。

而 log-blackhole 就是为这个需求而产生的。

**接管程序的输出**
```shell
$ my_program_a | grep error | log-blackhole my_program_error.log &
$ my_program_b | log-blackhole my_program.log &
```
**使用管道的日志**
```shell
$ tail -f my_program_error.log 
``` 

## 特性
- 依赖：linux , python 2.7 +
- 以Linux管道的方式往此程序里面写的数据，将直接丢弃，但会跟踪一个大小为0的日志文件的使用者
- 如果这个被跟踪的日志文件有使用者，那么往 log-blackhole 里传入的日志将写入这个跟踪日志文件，否则丢弃
- 当跟踪文件不再被使用的时候，这个日志跟踪文件又会切换成0字节

## 使用方法
**`your_program_output | your/path/of/log-blackhole [your_program_output.log] &`**
- `your_program_output`: 你的程序输出的信息
- `your/path/of/log-blackhole`: log-blackhole 所在的文件位置
- `your_program_output.log`: 可选的日志跟踪文件，如果未指定，那么它将输出为 log-blackhole 所在的目录.log 文件

## Example
```shell
$ yes | ./log-blackhole &
$ tail -f ./log-blackhole.log
```
