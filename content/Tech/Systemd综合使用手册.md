# Systemd综合使用手册
## 配置文件目录
读取目录：`/etc/systemd/system/`
rpm包安装目录：`/usr/lib/systemd/system/`
## 系统信息
### 电源操作
```bash
# 重启系统；
$ sudo systemctl reboot

# 关闭系统，切断电源；
$ sudo systemctl poweroff

# CPU停止工作；
$ sudo systemctl halt

# 暂停系统；
$ sudo systemctl suspend

# 让系统进入冬眠状态；
$ sudo systemctl hibernate

# 让系统进入交互式休眠状态；
$ sudo systemctl hybrid-sleep

# 启动进入救援状态（单用户状态）；
$ sudo systemctl rescue
```

### 系统状态
```bash
# 查看启动耗时；
$ systemd-analyze                                                                                       

# 查看每个服务的启动耗时；
$ systemd-analyze blame

# 显示瀑布状的启动过程流；
$ systemd-analyze critical-chain

# 显示指定服务的启动流；
$ systemd-analyze critical-chain atd.unit-type

# 显示系统状态；
$ systemctl status
```

### 基础信息
```bash
# 显示当前主机的基础信息；
$ hostnamectl

# 设置主机名；
$ sudo hostnamectl set-hostname rhel7

# 查看本地化设置；
$ localectl

# 设置本地化参数；
$ sudo localectl set-locale LANG=en_GB.utf8
$ sudo localectl set-keymap en_GB

# 查看当前时区设置；
$ timedatectl

# 显示所有可用的时区；
$ timedatectl list-timezones                                                                                   

# 设置当前时区；
$ sudo timedatectl set-timezone America/New_York
$ sudo timedatectl set-time YYYY-MM-DD
$ sudo timedatectl set-time HH:MM:SS
```

### 用户信息
```bash
# 列出当前session；
$ loginctl list-sessions

# 列出当前登录用户；
$ loginctl list-users

# 列出显示指定用户的信息；
$ loginctl show-user ruanyf
```

### 日志信息
```bash
# 查看日志（默认情况下 ，只保存本次启动的日志）；
$ sudo journalctl <服务/脚本/进程（通过_PID=*方式指定）/用户（通过_UID=*方式指定）路径>
# 参数：
    1.筛选
        -k 仅查看内核日志
        -b -* 查看本次启动前序启动的日志（本次为0，向前倒推数字递增）（默认为0，即本次）（需更改配置文件使其保存前序日志）
        -u application.unit-type 查看某个Unit的日志
        -p * 查看指定优先级的日志，可用数字/代号指定
            优先级：
                0: emerg
                1: alert
                2: crit
                3: err
                4: warning
                5: notice
                6: info
                7: debug
        --since=*;--until=* 查看指定时间的日志，
            可接受的时间格式：
                "2020-12-31 23:59:59"
                "20 min ago"
                09:00
                yesterday
    2.显示方式
        -n * 显示尾部的*行日志（如未指定，默认为10）
        -f 实时滚动显示最新日志
    3.输出方式
        --no-pager 获取无分页的标准化输出，可结合管道符使用
        -o 指定输出格式，如json、json-pretty等
    4.服务配置
        --disk-usage 显示日志占据的硬盘空间
        --vacuum-size=* 指定日志文件占据的最大空间
        --vacuum-time=* 指定日志文件保存多久
```

## Unit

### 系统资源
在systemd框架下，所有系统资源都被抽象成了“单元”（Unit），分12个大类：

1. 硬件实体：
    Device Unit：硬件设备
    Mount Unit：文件系统的挂载点
    Automount Unit：自动挂载点
2. 文件实体：
    Path Unit：文件或路径
    Socket Unit：进程间通信的 socket
    Swap Unit：swap 文件
3. 服务实体：
    Service unit：系统服务
    Slice Unit：进程组
    Target unit：多个Unit构成的一个组，一般为一个运行状态
    Scope Unit：不是由 Systemd 启动的外部进程
4. 系统管理：
    Snapshot Unit：Systemd 快照，可以切回某个快照
    Timer Unit：定时器

### 单元操作
#### 获取信息
```bash
# 列出正在运行的 Unit；
$ systemctl list-units
# 参数：
    --all 列出所有，包括没有找到配置文件的或启动失败的
    --failed 筛选加载失败的
    --state=* 筛选处于某状态的
    --type=* 筛选为某类型的

# 显示单个Unit的状态；
$ systemctl status application.unit-type
# 参数：
    -H 连接远程主机，后跟SSH连接信息，必须紧跟主命令
# 输出信息解读：
    Loaded行：配置文件的位置，是否设为开机启动
    Active行：表示正在运行
    Main PID行：主进程ID
    Status行：由应用本身（这里是 httpd ）提供的软件当前状态
    CGroup块：应用的所有子进程
    日志块：应用的日志

# 显示某个Unit是否正在运行/是否加载失败/是否建立了启动链接；
$ systemctl is-active/is-failed/is-enabled application.unit-type

# 显示某个Unit的所有底层参数；
$ systemctl show application.unit-type
# 参数：
    -p 指定一个显示属性，须紧跟'show'

# 显示某个Unit的依赖关系；
$ systemctl list-dependencies nginx.service
# 参数：
    --all 显示Target类依赖关系
```

#### 单元操作
```bash
# 立即启动/停止/重启一个Unit；
$ sudo systemctl start/stop/restart application.unit-type

# 杀死一个服务的所有子进程；
$ sudo systemctl kill application.unit-type

# 重新加载一个服务的配置文件；
$ sudo systemctl reload apache.unit-type

# 重载所有修改过的配置文件；
$ sudo systemctl daemon-reload

# 设置某个Unit的指定属性；
$ sudo systemctl set-property application.unit-type A=x
```

#### Target
```bash
# 查看启动时的默认 Target；
$ systemctl get-default

# 设置启动时的默认 Target；
$ sudo systemctl set-default multi-user.target

# 切换 Target 时，默认不关闭前一个 Target 启动的进程，systemctl isolate 命令改变这种行为；
# 关闭前一个 Target 里面所有不属于后一个 Target 的进程；
$ sudo systemctl isolate multi-user.target
```

## 配置文件
### 配置文件信息
```bash
# 列出所有配置文件；
$ systemctl list-unit-files
# 参数：
    --type=* 指定列出的配置文件的Unit类型
# 输出状态：
    enabled：已建立启动链接
    disabled：没建立启动链接
    static：该配置文件没有[Install]部分（无法执行），只能作为其他配置文件的依赖
    masked：该配置文件被禁止建立启动链接

# 读取配置文件内容；
systemctl cat application.unit-type
```
### 配置文件组成
#### 基本规则
配置文件分成几个区块。
每个区块的第一行，是用方括号表示的区块名，比如[Unit]。配置文件的区块名和字段名，都是大小写敏感的。
每个区块内部是一些等号连接的键值对，其等号两侧不能有空格。
#### 区块内容
##### `[Unit]`
[Unit]区块通常是配置文件的第一个区块，用来定义 Unit 的元数据，以及配置与其他 Unit 的关系。

1. 元数据
    Description：简短描述；
    Documentation：文档地址；
2. 依赖关系
    1. 需求/冲突关系
        Condition...：当前 Unit 运行必须满足的条件，否则不会运行；
        Conflicts：这里指定的 Unit 不能与当前 Unit 同时运行；
        Assert...：当前 Unit 运行必须满足的条件，否则会报启动失败；
    2. 强依赖
        Requires：当前 Unit 依赖的其他 Unit，如果它们没有运行，当前 Unit 会启动失败；
        Wants：与当前 Unit 配合的其他 Unit，如果它们没有运行，当前 Unit 不会启动失败；
        BindsTo：与Requires类似，它指定的 Unit 如果退出，会导致当前 Unit 停止运行；
    3. 弱依赖
        Before：如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之后启动；
        After：如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之前启动；

##### `[Install]`
[Install]通常是配置文件的最后一个区块，用来定义如何启动，以及是否开机启动。

1. 元数据
    Alias：当前 Unit 可用于启动的别名；
2. 依赖关系
    WantedBy：它的值是一个或多个 Target，当前 Unit 激活时（enable）符号链接会放入/etc/systemd/system目录下面以 Target 名 + .wants后缀构成的子目录中（强依赖）；
    RequiredBy：它的值是一个或多个 Target，当前 Unit 激活时，符号链接会放入/etc/systemd/system目录下面以 Target 名 + .required后缀构成的子目录中（弱依赖）；
3. 启动关系
    Also：当前 Unit 激活（enable）时，会被同时激活的其他 Unit；


##### `[Service]`
[Service]区块用来 Service 的配置，只有 Service 类型的 Unit 才有这个区块。
1. 进程类型
    Type：定义启动时的进程行为；值：
> simple（默认值）：ExecStart字段启动的进程为主进程；
> forking：ExecStart字段将以fork()方式启动，此时父进程将会退出，子进程将成为主进程；
> oneshot：类似于simple，但只执行一次，Systemd 会等它执行完，才启动其他服务；
> dbus：类似于simple，但会等待 D-Bus 信号后启动；
> notify：类似于simple，启动结束后会发出通知信号，然后 Systemd 再启动其他服务；
> idle：类似于simple，但是要等到其他任务都执行完，才会启动该服务。一种使用场合是为让该服务的输出，不与其他服务的输出相混合；

2. 执行指令
    *所有的启动设置之前，都可以加上一个连词号（-），表示"抑制错误"，即发生错误的时候，不影响其他命令的执行。*
    ExecStart：启动当前服务的命令；
    ExecStartPre：启动当前服务之前执行的命令；
    ExecStartPost：启动当前服务之后执行的命令；
    ExecReload：重启当前服务时执行的命令；
    ExecStop：停止当前服务时执行的命令；
    ExecStopPost：停止当其服务之后执行的命令；

    RuntimeDirectory：指定运行当前服务时的执行目录；
3. 异常响应
    RestartSec：自动重启当前服务间隔的秒数；
    TimeoutSec：定义 Systemd 停止当前服务之前等待的秒数；
4. 操作处理
    KillMode：定义Systemd如何停止当前服务；值：
    
> control-group（默认值）：当前控制组里面的所有子进程，都会被杀掉；
> process：只杀主进程；
> mixed：主进程将收到 SIGTERM 信号，子进程收到 SIGKILL 信号；
> none：没有进程会被杀掉，只是执行服务的 stop 命令；

   Restart：定义何种情况 Systemd 会自动重启当前服务；值：

> no（默认值）：退出后不会重启；
> on-success：只有正常退出时（退出状态码为0），才会重启；
> on-failure：非正常退出时（退出状态码非0），包括被信号终止和超时，才会重启；
> on-abnormal：只有被信号终止和超时，才会重启；
> on-abort：只有在收到没有捕捉到的信号终止时，才会重启；
> on-watchdog：超时退出，才会重启；
> always：不管是什么退出原因，总是重启；
> 
5. 元数据
    Environment：指定环境变量；

