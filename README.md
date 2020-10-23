# Ansible

> Ansible是基于python语言开放，只需在一台普通的服务器上运行即可，不需要在客户端服务器上安装客户端。因为Ansible是基于SSH 远程管理，而Linux 服务器大都离不开SSH，所以Ansible 不需要为配置工作添加额外的支持。

> Ansible 安装使用非常简单，而且基于上千个插件和模块，实现各种软件、平台、版本的管理，支持虚拟容器多层级的部署。

Ansible 自动化运维管理工具优点：
* 轻量级，更新时只需要在操作机上进行一次更新即可
* 采用SSH协议
* 不需要客户端安装agent
* 批量任务执行可以写成脚本，而且不用分发到远程客户端
* 使用python 编写，维护更简单
* 支持sudo 普通用户命令
* 去中心化管理

## Ansible 安装配置
> Ansible 可以工作在linux / BSD / Mac OS X等平台，对python 环境版本最低要求为python 2.6以上。安装之后默认主目录为 /etc/ansible/，其中hosts文件为被管理主机，ansible.cfg为ansible主配置文件，roles为角色或插件路径。

### Ansible 安装

#### 从Github获取Ansible
```
$ git clone git://github.com/ansible/ansible.git --recursive
$ cd ./ansible
```

#### 通过 pip 安装最新发布版本
```
$ sudo easy_install pip 
$ sudo pip install ansible
```

#### 通过apt 安装（ubuntu）
```
sudo apt-get install ansible
```
> 如果提示资源问题，则执行以下命令
```
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
```
### 配置参数
> ansible 默认配置文件为 /etc/ansible/ansible.cfg，配置文件中可以对Ansible进行各项参数的调整，包括并发线程、用户、模块路径、配置优化等。
#### ansible.cfg
* [defaults]
> 日常可能用到的配置，这些多数保持默认即可。
```
[defaults]
inventory = /etc/ansible/hosts                                           #定义Inventory
library =  /usr/share/my_modules/                                   #自定义lib库存放目录 
remote_tmp = $HOME/.ansible/tmp                               #临时文件远程主机存放目录 
local_tmp  =  $HOME/.ansible/tmp                                #临时文件本地存放目录 
forks = 5                                                                        #默认开启的并发数 
poll_interval   =  15                                                       #默认轮询时间间隔 
sudo_user   =   root                                                      #默认sudo用户 
ask_sudo_pass = True                                               #是否需要sudo密码 
ask_pass = True                                                        #是否需要密码
roles_path   =  /etc/ansible/roles                                #默认下载的Roles存放的目录 
host_key_checking  = False                                     #首次连接是否需要检查key认证，建议设为False
timeout   =  10                                                          #默认超时时间
remote_user   =  root                                                         #如没有指定用户，默认使用的远程连接用户 
log_path =  /var/log/ansible.log                               #执行日志存放目录 
module_name  =  command                                  #默认执行的模块 
action_plugins =  /usr/share/ansible/plugins/action              #action插件的存放目录 
callback_plugins =  /usr/share/ansible/plugins/callback       #callback插件的存放目录 
connection_plugins  =  /usr/share/ansible/plugins/connection   #connection插件的存放目录 
lookup_plugins =  /usr/share/ansible/plugins/lookup       #lookup插件的存放目录 
vars_plugins  =  /usr/share/ansible/plugins/vars            #vars插件的存放目录 
filter_plugins  =  /usr/share/ansible/plugins/filter            #filter插件的存放目录 
test_plugins  =  /usr/share/ansible/plugins/test             #test插件的存放目录 
strategy_plugins  = /usr/share/ansible/plugins/strategy   #strategy插件的存放目录 
fact_caching  =  memory                           #getfact缓存的主机信息存放方式 
retry_files_enabled  = False                    
retry_files_save_path =  ~/.ansible-retry   #错误重启文件存放目录
```

* [privilege_escalation]
> 出于安全角度考虑，部分公司不希望直接以root的高级管理员权限直接部署应用，往往会开放普通用户权限并给予sudo的权限，该部分配置主要针对sudo用户提权的配置 
```
become = True         #是否sudo
become_method=sudo   #sudo 方式 
become_user=root     #sudo后变为root用户 
become_ask_pass=False   #sudo后是否验证密码
```

* [privilege_connection]
> 该配置不常用到
```
record_host_keys=False   #不记录新主机的key以提升效率
pty=False        #禁用sudo功能
```

* [ssh_connection]
> Ansible默认使用的是SSH协议进行与对端主机进行通信。但配置项较少，多数默认即可
```
pipelining = False
```

* [accelerate]
> Ansible连接加速相关配置。多数保持默认即可
```
accelerate_port  =  5099          #加速连接端口
accelerate_timeout  =  30        #命令执行超时时间，单位秒
accelerate_connect_timeout = 5.0   #连接超时时间，单位秒
accelerate_multi_key  =  yes    
```

* [selinux]
> 关于selinux的相关配置几乎不会涉及，保持默认配置即可
```
libvirt_lxc_noseclabel  =  yes
libvirt_lxc_noseclabel  =  yes 
```

* [colors]
> Ansible对于输出结果的颜色也进行了详尽的定义且可配置，保持默认即可
```
highlight  =  white
verbose  =  blue
warn   =  bright purple
error  =   red
debug  =  dark  gray
deprecate  =  purple
skip  =  cyan
unrechable  =  red
ok    =  green
changed  =  yellow
diff_add  =  green 
diff_remove  =  red
diff_lines   =  cyan
```

#### Inventory文件（hosts）
```
ansible_ssh_host
      将要连接的远程主机名.与你想要设定的主机的别名不同的话,可通过此变量设置.

ansible_ssh_port
      ssh端口号.如果不是默认的端口号,通过此变量设置.

ansible_ssh_user
      默认的 ssh 用户名

ansible_ssh_pass
      ssh 密码(这种方式并不安全,我们强烈建议使用 --ask-pass 或 SSH 密钥)

ansible_sudo_pass
      sudo 密码(这种方式并不安全,我们强烈建议使用 --ask-sudo-pass)

ansible_sudo_exe (new in version 1.8)
      sudo 命令路径(适用于1.8及以上版本)

ansible_connection
      与主机的连接类型.比如:local, ssh 或者 paramiko. Ansible 1.2 以前默认使用 paramiko.1.2 以后默认使用 'smart','smart' 方式会根据是否支持 ControlPersist, 来判断'ssh' 方式是否可行.

ansible_ssh_private_key_file
      ssh 使用的私钥文件.适用于有多个密钥,而你不想使用 SSH 代理的情况.

ansible_shell_type
      目标系统的shell类型.默认情况下,命令的执行使用 'sh' 语法,可设置为 'csh' 或 'fish'.

ansible_python_interpreter
      目标主机的 python 路径.适用于的情况: 系统中有多个 Python, 或者命令路径不是"/usr/bin/python",比如  \*BSD, 或者 /usr/bin/python
      不是 2.X 版本的 Python.我们不使用 "/usr/bin/env" 机制,因为这要求远程用户的路径设置正确,且要求 "python" 可执行程序名不可为 python以外的名字(实际有可能名为python26).

      与 ansible_python_interpreter 的工作方式相同,可设定如 ruby 或 perl 的路径....
```
### ansible常用模块
#### 主机连通性
使用ansible web -m ping命令来进行主机连通性测试
* 实例
```
ansible web -m ping
# 其中web为host组
```
#### command模块
直接在远程主机上执行命令，并将结果返回本主机。
* 实例
```
ansible web -m command -a 'ss -ntl'
```
#### shell模块
可以在远程主机上调用shell解释器运行命令，支持shell的各种功能，例如管道等。
* 实例
```
ansible web -m shell -a 'cat /etc/passwd |grep "keer"'
```
#### copy模块
用于将文件复制到远程主机，同时支持给定内容生成文件和修改权限等。
* 复制文件
    * 实例
```
ansible web -m copy -a 'src=~/hello dest=/data/hello'
```
* 给定内容生成文件，并制定权限
    * 实例
```
ansible web -m copy -a 'content="I am keer\n" dest=/data/name mode=666'
```
* 覆盖
    * 实例
```
ansible web -m copy -a 'content="I am keerya\n" backup=yes dest=/data/name mode=666'
```
### file模块
用于设置文件的属性，比如创建文件、创建链接文件、删除文件等。
``` json
force　　#需要在两种情况下强制创建软链接，一种是源文件不存在，但之后会建立的情况下；另一种是目标软链接已存在，需要先取消之前的软链，然后创建新的软链，有两个选项：yes|no
group　　#定义文件/目录的属组。后面可以加上mode：定义文件/目录的权限
owner　　#定义文件/目录的属主。后面必须跟上path：定义文件/目录的路径
recurse　　#递归设置文件的属性，只对目录有效，后面跟上src：被链接的源文件路径，只应用于state=link的情况
dest　　#被链接到的路径，只应用于state=link的情况
state　　#状态，有以下选项：

    directory: #如果目录不存在，就创建目录
    file: #即使文件不存在，也不会被创建
    link: #创建软链接
    hard: #创建硬链接
    touch: #如果文件不存在，则会创建一个新的文件，如果文件或目录已存在，则更新其最后修改时间
    absent: #删除目录、文件或者取消链接文件
```

* 创建目录
    * 实例
```
ansible web -m file -a 'path=/data/app state=directory'
```
* 创建链接文件
    * 实例
```
ansible web -m file -a 'path=/data/bbb.jpg src=aaa.jpg state=link'
```
* 删除文件
    * 实例
```
ansible web -m file -a 'path=/data/a state=absent'
```
#### fetch模块
用于从远程某主机获取（复制）文件到本地。
```json
dest：用来存放文件的目录
src：在远程拉取的文件，并且必须是一个file，不能是目录
```
* 实例
```
ansible web -m fetch -a "src=/app/f1 dest=/app/"
```
#### cron模块
管理cron计划任务；-a "": 设置管理节点生成定时任务
```
action: 
cron backup=   #如果设置，创建一个crontab备份 【yes|no】
cron_file=    #如果指定, 使用这个文件cron.d，而不是单个用户
crontab
　　day=     #日应该运行的工作( 1-31, *, */2, )
　　hour=   # 小时 ( 0-23, *, */2, )
　　minute=    #分钟( 0-59, *, */2, )
　　month=     #月( 1-12, *, /2, )
　　weekday   # 周 ( 0-6 for Sunday-Saturday,, )
　　job=       #指明运行的命令是什么
　　name=   #定时任务描述
　　reboot    # 任务在重启时运行，不建议使用，建议使用special_time
　　special_time   #特殊的时间范围，参数：reboot（重启时）,annually（每年）,monthly（每月）,weekly（每周）,daily（每天）,hourly（每小时）
　　state   #指定状态，present表示添加定时任务，也是默认设置，absent表示删除定时任务
　　user    #以哪个用户的身份执行
```
* 实例
```
ansible web -m cron -a "name='Clear the iptable' minute=*/5 job='/sbin/iptables -F'"
```
删除计划任务
```
ansible web -m cron -a "name='Clear the iptable' minute=*/5 job='/sbin/iptables -F' state=absent" 
```
#### yum模块
包管理
```
conf_file    #设定远程yum安装时所依赖的配置文件。如配置文件没有在默认的位置。
disable_gpg_check   #是否禁止GPG checking，只用于`present‘ or `latest’。
disablerepo   #临时禁止使用yum库。 只用于安装或更新时。
enablerepo   #临时使用的yum库。只用于安装或更新时。
name=    #所安装的包的名称
state=     #present安装， latest安装最新的, absent 卸载软件。
update_cache    #强制更新yum的缓存。
```
* 实例
```
ansible web -m yum -a "name=dstat state=present disable_gpg_check=yes"
```
```
ansible web -m yum -a "name=dstat state=absent"
```
#### service模块
服务程序管理
```
arguments   #命令行提供额外的参数
enabled   #设置开机启动。
name=     #服务名称
runlevel    #开机启动的级别，一般不用指定。
sleep    #在重启服务的过程中，是否等待。如在服务关闭以后等待2秒再启动。
state     #started启动服务， stopped停止服务， restarted重启服务， reloaded重载配置
```
* 实例
```
开启远程被控制端的nginx 服务
ansible web -m service -a "name=nginx state=started"
```
```
关闭远程的nginx 服务
ansible web -m service -a "name=nginx state=stopped"
```
#### user模块
用户模块,管理用户帐号
```
comment        # 用户的描述信息
createhome    # 是否创建家目录
force      # 在使用state=absent是, 行为与userdel -force一致.
group     # 指定基本组
groups   # 指定附加组，如果指定为(groups=)表示删除所有组
home     # 指定用户家目录
move_home    # 如果设置为home=时, 试图将用户主目录移动到指定的目录
name     # 指定用户名
non_unique     # 该选项允许改变非唯一的用户ID值
password       # 指定用户密码，若指定的是明文密码，是不能用的，需用md5加密过后的密码
remove   # 在使用state=absent时, 行为是与userdel -remove一致
shell      # 指定默认shell
state      # 设置帐号状态，不指定为创建，指定值为absent表示删除
system  # 当创建一个用户，设置这个用户是系统用户。这个设置不能更改现有用户
uid     # 指定用户的uid
update_password    # 更新用户密码
```
* 实例
```
创建用户along01，uid=1111，家目录在/app/along01 下
ansible web -m user -a 'name=along01 comment="along01 is along" uid=1111 group=along shell=/bin/bash home=/app/along01'
ansible web -m shell -a "cat /etc/passwd |grep along01" 查看
ansible web -m user -a "name=along01 state=absent" 删除用户
```
#### group模块
用户组模块,添加或删除组
```
gid         # 设置组的GID号
name=  # 管理组的名称
state     # 指定组状态，默认为创建，设置值为absent为删除
system  # 设置值为yes，表示为创建系统组
```
* 实例
```
ansible web -m group -a 'name=tom state=present'
```
#### script模块
在指定节点运行服务端的脚本
* 实例
```
ansible web -m script -a '/app/test.sh' 在远程被控制的机器执行脚本
```
#### setup模块
查机器的所有facts信息
* facts组件是Ansible用于采集被管机器设备信息的一个功能，我们可以使用setup模块查机器的所有facts信息，可以使用filter来查看指定信息。整个facts信息被包装在一个JSON格式的数据结构中，ansible_facts是最上层的值。
* facts就是变量，内建变量 。每个主机的各种信息，cpu颗数、内存大小等。会存在facts中的某个变量中。调用后返回很多对应主机的信息，在后面的操作中可以根据不同的信息来做不同的操作。如redhat系列用yum安装，而debian系列用apt来安装软件。
* setup模块，主要用于获取主机信息，在playbooks里经常会用到的一个参数gather_facts就与该模块相关。
* setup模块下经常使用的一个参数是filter 参数，查询的是全部信息，很多，filter 相当于匹配筛选。

```
ansible web -m setup -a "filter='*mem*'" 查看内存的信息
```
### playbook模块
#### 介绍
* playbook是ansible用于配置，部署，和管理被控节点的剧本。
* 通过playbook的详细描述，执行其中的一系列tasks，可以让远端主机达到预期的状态。playbook就像Ansible控制器给被控节点列出的的一系列to-do-list，而被控节点必须要完成。
* 也可以这么理解，playbook 字面意思，即剧本，现实中由演员按照剧本表演，在Ansible中，这次由计算机进行表演，由计算机安装，部署应用，提供对外服务，以及组织计算机处理各种各样的事情

Ansible playbook使用场景
* 执行一些简单的任务，使用ad-hoc命令可以方便的解决问题，但是有时一个设施过于复杂，需要大量的操作时候，执行的ad-hoc命令是不适合的，这时最好使用playbook。
* 就像执行shell命令与写shell脚本一样，也可以理解为批处理任务，不过playbook有自己的语法格式。
* 使用playbook你可以方便的重用这些代码，可以移植到不同的机器上面，像函数一样，最大化的利用代码。在你使用Ansible的过程中，你也会发现，你所处理的大部分操作都是编写playbook。可以把常见的应用都编写成playbook，之后管理服务器会变得十分简单。
#### playbook格式




### 附录
#### Linux常用命令
- ssh-keygen

```
$ssh-keygen -t rsa -C "huangt" -f aliyun.key

$vim /etc/ssh/sshd_config

RSAAuthentication yes
PubkeyAuthentication yes

PermitRootLogin no //禁止root登录
PasswordAuthentication yes //允许密码登录，根据你的情况设置
```
