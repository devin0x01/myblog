# 1.常用命令
> 选项：
> -c:执行脚本前先执行的命令，可多次使用。
> -d:debug模式，可以在运行时输出一些诊断信息，与在脚本开始处使用exp_internal 1相似。
> -D:启用交换调式器,可设一整数参数。
> -f:从文件读取命令，仅用于使用#!时。如果文件名为"-"，则从stdin读取(使用"./-"从文件名为-的文件读取)。
> -i:交互式输入命令，使用"exit"或"EOF"退出输入状态。
> --:标示选项结束(如果你需要传递与expect选项相似的参数给脚本时)，可放到#!行:#!/usr/bin/expect --。
> -v:显示expect版本信息。

```shell
# 命令行参数 
# $argv，参数数组，使用[lindex $argv n]获取，$argv 0为脚本名字
# $argc，参数个数
set username [lindex $argv 1]  # 获取第1个参数
set passwd [lindex $argv 2]    # 获取第2个参数
 
set timeout 30 # 设置超时
 
# spawn是expect内部命令，开启ssh连接
spawn ssh -l username 192.168.1.1
 
# 判断上次输出结果里是否包含“password:”的字符串，如果有则立即返回，否则就等待一段时间(timeout)后返回
expect "password:"
 
# 发送内容ispass(密码、命令等)
send "ispass\r"
 
# 发送内容给用户
send_user "$argv0 [lrange $argv 0 2]\n"
send_user "It's OK\r"
# 执行完成后保持交互状态，控制权交给控制台(手工操作)。否则会完成后会退出。
interact
```
# 2.例子
## 2.1自动telnet会话
```shell
#!/usr/bin/expect -f
set ip [lindex $argv 0 ]         # 接收第1个参数,作为IP
set userid [lindex $argv 1 ]     # 接收第2个参数,作为userid
set mypassword [lindex $argv 2 ] # 接收第3个参数,作为密码
set mycommand [lindex $argv 3 ]  # 接收第4个参数，作为命令
set timeout 10                   # 设置超时时间
 
# 向远程服务器请求打开一个telnet会话，并等待服务器询问用户名
spawn telnet $ip
    expect "username:"
    # 输入用户名，并等待服务器询问密码
    send "$userid\r"
    expect "password:"
    # 输入密码，并等待键入需要运行的命令
    send "$mypassword\r"
    expect "%"
    # 输入预先定好的密码，等待运行结果
    send "$mycommand\r"
    expect "%"
    # 将运行结果存入到变量中，显示出来或者写到磁盘中
    set results $expect_out(buffer)
    # 退出telnet会话，等待服务器的退出提示EOF
    send "exit\r"
    expect eof
```
## 2.2其他
```shell
#!/usr/bin/expect

expect {
	"Are you sure you want to continue connecting (yes/no)?" {send "yes\r"; exp_continue}
  "Password" {send "${PWD}\r"}
}

expect "sftp>" {send "get /home/tools/123.zip\r"}
expect "sftp>" {send "quit\r"}

expect eof
EOF
```
# 参考文献

1. expect - 自动交互脚本：[https://xstarcd.github.io/wiki/shell/expect.html](https://xstarcd.github.io/wiki/shell/expect.html)
