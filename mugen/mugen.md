# mugen

## 简述

* mugen 是 openEuler 社区开源的自动化测试框架

* 使用 Python3 和 Shell 这两种脚本语言编写

* openEuler 版本迭代可以通过 mugen 进行测试 产出的错误得到解决可以帮助完善测试用例与修复软件包 从而保证系统的稳定性与安全性

## mugen 的基本使用方法

### 需要配置环境

* 需要是使用 rpm/yum/dnf/ 包管理器的 Linux 发行版
* 有的测试脚本会使用到 oe 自带的 config 变量
* 官方仓库：https://gitee.com/openeuler/mugen

### 安装与配置运行

1. 拉取源码 `git clone https://gitee.com/openeuler/mugen.git`
2. `cd mugen`
3. 安装依赖 `./dep_install.sh`
4. 首次运行以配置环境变量

`bash mugen.sh -c --ip $ip --password $passwd --user $user --port $port`

备注：$ip 测试机器的 ip 地址； $passwd 密码； $user 用户名 默认是 root； $port 端口 默认是22

* 配置文件存储在 `./conf/env.json`

示例：以下是我测试环境的配置文件

```
[root@10 conf]# cat env.json 
{
    "NODE": [
        {
            "ID": 1,
            "LOCALTION": "local",
            "MACHINE": "physical",
            "IPV6": "fec0::b1b2:a524:ee47:9185",
            "FRAME": "riscv64",
            "NIC": "eth0",
            "MAC": "52:54:00:12:34:56",
            "IPV4": "10.0.2.15",
            "USER": "root",
            "PASSWORD": "openEuler12#$",
            "SSH_PORT": 12055,
            "BMC_IP": "",
            "BMC_USER": "",
            "BMC_PASSWORD": ""
        }
    ]
}
```

### 测试机

每一个测试机都是一个 `NODE`

如果需要进行远程执行，可以通过 `-s` 连接到远程的 `NODE` 再执行测试 

### 基本参数

*示例在文档后半实操部分*

* `-a` 运行所有测试用例
* `-f` 运行测试套
* `-r` 运行一个测试用例
* `-c` 配置测试环境
* `-x` 以 debug 模式运行
* `-b` 如果测试套含有 Makefile 执行 make
* `-s` 在远程机器执行用例
* `-m` 调试配置文件

### mugen 的运行流程

1. 运行主程序 `mugen.sh`
2. 调用测试套 (在 test2cases 里面有 json 文件定义：见文档下一模块)
3. 检查测试环境是否符合测试套要求：否(skip)； 是(进行下一步)
4. 调用 testcases
5. 检查测试环境是否符合测试用例要求：是(进行测试，返回结果)； 否(skip)

备注：测试结果保存在当前目录下的 `results` 文件夹； 执行日志保存在当前目录下的 `logs` 文件夹

以上是 mugen 工具执行测试的基本流程(详细的执行流程在后边分析)

### 测试套定义

* 在运行流程中提到了第二阶段要调用测试套，于是需要读取定义测试套的 json 文件

示例(这个是 pcp 测试套的 json 文件的部分内容)：

```
{
    "path": "$OET_PATH/testcases/cli-test/pcp", 		*#描述测试套所在的目录路径*
    "machine num": 2,                                   *#需要两个 NODE*
    "cases": [
        {
            "name": "oe_test_dbpmda"                    *#测试脚本*
        },
        {
            "name": "oe_test_pcp"
        },
}
```

* 在运行前需要检查环境是否符合 json 里定义的要求 额外硬件设备支持定义的方法见下方：

1. 多磁盘支持：

摘自 admin-guide 测试套

```
        {
            "name": "oe_test_lvm_pvdispay",
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },

```

这段代码，指的就是需要额外四块磁盘的支持，每块磁盘不小于 2GB

可以直接向虚拟机中添加虚拟磁盘

2. 多网卡和多设备：

摘自 pcp 测试套(多设备)

```
        {
            "name": "oe_test_pcp_pcp-import-ganglia2pcp",
            "machine num": 2
        },
```

这里的 `"machine num": 2` 指的就是需要两个 NODE

摘自 network-basic.json (多网卡)

```
    "path": "${OET_PATH}/testcases/system-test/network-test/network_basic",
    "machine num": 2,
    "add network interface": 1,
```

这里的 `"add network interface": 1` 指的就是需要额外的网络接口

3. 设定测试机必须为物理机

`"machine type": "physical"` 可以声明测试机必须为物理机，也可以填写 `kvm` 设置为虚拟机

### 获取网卡信息

[参考自赵老师的文档](https://github.com/openeuler-riscv/oerv-team/blob/main/cases/2024.12.25-mugen%E5%85%A5%E9%97%A8-%E8%B5%B5%E9%A3%9E%E6%89%AC.md#json_describe)

以下包含了我对这段脚本的学习认识(注释中)：

```
if os.environ.get("NODE" + str(node) + "_LOCALTION") == "local":	#判断是否是本地 NODE	
    dev_info = subprocess.getoutput("cat /proc/device-tree/model")  #获取设备信息
    if "Lichee Pi 4A" in dev_info:                                  #如果是 荔枝派4a 就进行特殊处理
        output = subprocess.getoutput(
            "ip addr show | grep '^[0-9]:'|grep -v lo |grep -v virbr |grep -v wlan0 |grep -v dummy0 |grep -v sit0@NONE |grep -v ip_vti0@NONE |cut -d ':' -f2 |sed -e 's/^[[:space:]]*//' | grep -v '"
            + os.environ.get("NODE" + str(node) + "_NIC")
            + "'"
        ).replace("\n", " ")
    else:                                                           #非 荔枝派4a 就直接使用 `lshw` 获取网卡信息
        output = subprocess.getoutput(
            "lshw -class network | grep -A 5 'description: Ethernet interface' | grep 'logical name:' | awk '{print $NF}' | grep -v '"
            + os.environ.get("NODE" + str(node) + "_NIC")
            + "'"
        ).replace("\n", " ")
```

[添加测试套的规范要求](https://gitee.com/openeuler/mugen/blob/master/doc/%E6%B5%8B%E8%AF%95%E7%94%A8%E4%BE%8B%E6%A3%80%E8%A7%86%E8%A7%84%E8%8C%83.md)

分隔线------------

## 往下是 mugen 的架构与用例脚本分析 (参考自赵老师的文档)

```
[root@10 mugen]# ls
check_casename.py  conf		   libs      qemu_ctl.sh   suite2cases
combination	   dep_install.sh  License   README.en.md  testcases
combination.sh	   doc		   mugen.sh  runoet.sh
```

* `mugen.sh` 主程序脚本
* `libs` mugen 的库函数
* `suite2cases` 定义测试套的 json 文件
* `testcases` 测试用例脚本
* `results` 测试结果
* `logs` 测试日志
* `doc` 相关的规范文档

`testcases` 中的目录：

* cli-test 命令行测试
* doc-test linux 基础功能测试
* embedded-test 嵌入式测试
* feature-test openEuler 特性测试
* security_test 安全测试
* smoke-test 冒烟测试 可以快速检查软件的可用性
* system-test 系统测试

### 测试用例脚本分析

示例(admin-guide 测试套中的 oe_test_basic_add_user.sh)：

```
#!/usr/bin/bash

# Copyright (c) 2021. Huawei Technologies Co.,Ltd.ALL rights reserved.
# This program is licensed under Mulan PSL v2.
# You can use it according to the terms and conditions of the Mulan PSL v2.
#          http://license.coscl.org.cn/MulanPSL2
# THIS PROGRAM IS PROVIDED ON AN "AS IS" BASIS, WITHOUT WARRANTIES OF ANY KIND,
# EITHER EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO NON-INFRINGEMENT,
# MERCHANTABILITY OR FIT FOR A PARTICULAR PURPOSE.
# See the Mulan PSL v2 for more details.

# #############################################
# @Author    :   xuchunlin
# @Contact   :   xcl_job@163.com
# @Date      :   2020.04-09
# @License   :   Mulan PSL v2
# @Desc      :   Add User test
# ############################################
source ${OET_PATH}/libs/locallibs/common_lib.sh
function pre_test() {
    LOG_INFO "Start prepare the test environment!"
    user="testuser"
    grep -w "${user}" /etc/passwd && userdel -r "${user}"
    grep -w "${user}1" /etc/passwd && userdel -r "${user}1"
    groupdel ${user}
    LOG_INFO "End of prepare the test environment!"
}

function run_test() {
    LOG_INFO "Start executing testcase!"
    useradd ${user}
    useradd -e 2020-10-30 ${user}1
    grep -w "${user}" /etc/passwd
    CHECK_RESULT $?
    grep -w "${user}1" /etc/passwd
    CHECK_RESULT $?
    chage -l ${user}1 | grep 2020 | grep 30
    CHECK_RESULT $?
    chage -M 4 ${user}1
    chage -l ${user}1 | grep Maximum | grep 4
    useradd --help
    CHECK_RESULT $?
    LOG_INFO "End of testcase execution!"
}

function post_test() {
    LOG_INFO "start environment cleanup."
    userdel -r ${user}
    userdel -r ${user}1
    LOG_INFO "Finish environment cleanup."
}

main $@
```

这是一个用于测试 `useradd` 指令的测试脚本

`source ${OET_PATH}/libs/locallibs/common_lib.sh` 首先加载 mugen 的通用库，这个库中提供了常用的函数

* 函数 pre_test() 部分：

```
grep -w "${user}" /etc/passwd && userdel -r "${user}"
grep -w "${user}1" /etc/passwd && userdel -r "${user}1"
```

检查系统中是否已有同名用户，有则删除

* `LOG_INFO` 是输出日志

* 函数 run_test() 部分：

```
function run_test() {
    LOG_INFO "Start executing testcase!"
    useradd ${user}
    useradd -e 2020-10-30 ${user}1
    grep -w "${user}" /etc/passwd
    CHECK_RESULT $?
    grep -w "${user}1" /etc/passwd
    CHECK_RESULT $?
    chage -l ${user}1 | grep 2020 | grep 30
    CHECK_RESULT $?
    chage -M 4 ${user}1
    chage -l ${user}1 | grep Maximum | grep 4
    useradd --help
    CHECK_RESULT $?
    LOG_INFO "End of testcase execution!"
}
```

* 添加两个用户 并验证是否成功添加
* 验证帮助信息是否可用
* 设置用户过期时间 并验证此功能是否可用


`CHECK_RESULT $?` 用于检查上一条语句的返回值是否为 0 这里配合 `grep` 命令可以验证命令是否执行成功

* 函数 post_test() 部分：

```
function post_test() {
    LOG_INFO "start environment cleanup."
    userdel -r ${user}
    userdel -r ${user}1
    LOG_INFO "Finish environment cleanup."
}
```

开始清理环境

`userdel -r` 删除刚刚测试中创建的用户

`main $@` 主程序 调用 main 函数 也就是 pre_test() run_test() post_test()

* 这里的 main 函数是 通用库里的 main 函数

```
function main() {
    if [ -n "$(type -t post_test)" ]; then
        trap post_test EXIT INT HUP TERM || exit 1
    else
        trap POST_TEST_DEFAULT EXIT INT HUP TERM || exit 1
    fi

    if ! rpm -qa | grep expect >/dev/null 2>&1; then
        dnf -y install expect
    fi

    if [ -n "$(type -t config_params)" ]; then
        config_params
    fi

    if [ -n "$(type -t pre_test)" ]; then
        pre_test
    fi

    if [ -n "$(type -t run_test)" ]; then
        run_test
        CASE_RESULT $?
    fi
}
```

分析：

```
if [ -n "$(type -t post_test)" ]; then
        trap post_test EXIT INT HUP TERM || exit 1
else
        trap POST_TEST_DEFAULT EXIT INT HUP TERM || exit 1
fi
```

我个人的理解上来说，这段的作用就是：无论测试是否中途失败、被强制中断

都会自动调用 post_test 函数进行环境还原

备注：当收到 退出（EXIT）或 中断信号（INT/HUP/TERM）时执行函数对环境进行清理

这样可以保证测试环境的安全稳定

```
if ! rpm -qa | grep expect >/dev/null 2>&1; then
        dnf -y install expect
fi
```

验证 except 是否安装

```
    if [ -n "$(type -t config_params)" ]; then
        config_params
    fi
```

检查是否有 config_params

用于自定义参数初始化，比如设置环境变量、全局变量等

```
    if [ -n "$(type -t pre_test)" ]; then
        pre_test
    fi
```

执行 pre_test 函数

执行测试前的准备工作

```
    if [ -n "$(type -t run_test)" ]; then
        run_test
        CASE_RESULT $?
    fi
```

这个就是执行测试进程 从这里开始执行测试了

并将结果传给 CASE_RESULT 判断测试结果

备注：上面提到的 except 是一个命令行自动化工具

## mugen 测试错误的解读分析

### 大致有以下几种问题：

1. 测试用例问题

* 测试用例错误
* 未做 riscv 适配

2. 软件包问题

* 软件包存在 issue 
* 没有 riscv 下的包

3. 不适用 riscv

软件包或命令是非 riscv 平台特有的 通常是使用 exit code 255 退出 直接跳过该用例

### exit code 分析

通常有以下几种 exit code

* 0 代表测试用例执行成功
* 1-254 测试脚本执行失败 数字为未通过的次数
* 255 测试环境不满足要求 如上文提到的不适用 riscv 平台 无法进行测试

### CASE_RESULT 函数

```
function CASE_RESULT() {
    case_re=$1

    test -z "$exec_result" && {
        test "$case_re" -eq 0 && {
            LOG_INFO "succeed to execute the case."
            exec_result=""
            exit 0
        }
        LOG_ERROR "failed to execute the case."
        exit "$case_re"
    }

    test "$exec_result" -gt 0 && {
        LOG_ERROR "failed to execute the case."
        exit "$exec_result"
    }
    LOG_INFO "succeed to execute the case."
    exit "$exec_result"
}
```

这个函数就是检查 `exec_result` 的值 默认为 0 即没有出错

检查是否有 exec_result 变量 如果没有就接受程序本身的返回值（大概？这个我没有太看懂

### CHECK_RESULT 函数

```
function CHECK_RESULT() {
    actual_result=$1
    expect_result=${2-0}
    mode=${3-0}
    error_log=$4
    exit_mode=${5-0}

    if [ -z "$*" ]; then
        LOG_ERROR "Missing parameter error code."
	((exec_result++))
        return 1
    fi

    if [ "$mode" -eq 0 ]; then
        test "$actual_result"x != "$expect_result"x && {
            test -n "$error_log" && LOG_ERROR "$error_log"
            ((exec_result++))
            LOG_ERROR "${BASH_SOURCE[1]} line ${BASH_LINENO[0]}"
	    if [ "$exit_mode" -eq 1 ]; then
                    exit 1;
            fi
        }
    else
        test "$actual_result"x == "$expect_result"x && {
            test -n "$error_log" && LOG_ERROR "$error_log"
            ((exec_result++))
            LOG_ERROR "${BASH_SOURCE[1]} line ${BASH_LINENO[0]}"
	    if [ "$exit_mode" -eq 1 ]; then
                    exit 1;
            fi
        }
    fi

    return 0
}
```

这个是验证用例测试结果的重要函数

首先，确认参数是存在的，一个函数都没有是不被允许的

一个参数都没有的话就直接 exec_result++ 了

有两个 mode 一个是返回值和预期值相等为成功 另一个是返回值和预期值不相等为成功

运行完通过日志就可以很容易判断出错误的原因以及问题在哪里了

分隔线------------

### 接下来是服务类程序的判别方法 刚刚是命令行类程序

备注：刚刚分析的那个通用的脚本来自 `libs/locallibs/common_lib.sh`

接下来提到的来自 `testcases/cli-test/common/common_lib.sh`

```
#!/bin/bash

# Copyright (c) 2021. Huawei Technologies Co.,Ltd.ALL rights reserved.
# This program is licensed under Mulan PSL v2.
# You can use it according to the terms and conditions of the Mulan PSL v2.
#          http://license.coscl.org.cn/MulanPSL2
# THIS PROGRAM IS PROVIDED ON AN "AS IS" BASIS, WITHOUT WARRANTIES OF ANY KIND,
# EITHER EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO NON-INFRINGEMENT,
# MERCHANTABILITY OR FIT FOR A PARTICULAR PURPOSE.
# See the Mulan PSL v2 for more details.

# #############################################
# @Author    :   huangrong
# @Contact   :   1820463064@qq.com
# @Date      :   2020/11/03
# @License   :   Mulan PSL v2
# @Desc      :   common function library for service restart of the base images
# #############################################
# shellcheck disable=SC2086
source "${OET_PATH}/libs/locallibs/common_lib.sh"

function test_execution() {
    service=$1
    log_time="$(date '+%Y-%m-%d %T')"
    test_restart "${service}"
    test_enabled "${service}"
    journalctl --since "${log_time}" -u "${service}" | grep -i "fail\|error" | grep -v -i "DEBUG\|INFO\|WARNING"
    CHECK_RESULT $? 0 1 "There is an error message for the log of ${service}"
}

function test_restart() {
    service=$1
    systemctl restart "${service}"
    CHECK_RESULT $? 0 0 "${service} restart failed"
    SLEEP_WAIT 5
    systemctl status "${service}" | grep "Active: active"
    CHECK_RESULT $? 0 0 "${service} restart failed"
    systemctl stop "${service}"
    CHECK_RESULT $? 0 0 "${service} stop failed"
    SLEEP_WAIT 5
    systemctl status "${service}" | grep "Active: inactive"
    CHECK_RESULT $? 0 0 "${service} stop failed"
    systemctl start "${service}"
    CHECK_RESULT $? 0 0 "${service} start failed"
    SLEEP_WAIT 5
    systemctl status "${service}" | grep "Active: active"
    CHECK_RESULT $? 0 0 "${service} start failed"
}

function test_oneshot() {
    service=$1
    status=$2
    systemctl status "${service}" | grep "Active" | grep -v "${status}"
    CHECK_RESULT $? 0 1 "There is an error for the status of ${service}"
    test_enabled "${service}"
    journalctl -u "${service}" | grep -i "fail\|error" | grep -v -i "DEBUG\|INFO\|WARNING"
    CHECK_RESULT $? 0 1 "There is an error message for the log of ${service}"
}

function test_enabled() {
    service=$1
    state="$(systemctl is-enabled "${service}")"
    if [ "${state}" == "enabled" ]; then
	symlink_file=$(systemctl disable "${service}" 2>&1 | grep "Removed" | awk '{print $2}' | awk '{print substr($0,1,length($0)-1)}' | tr -d '"')
        find ${symlink_file}
        CHECK_RESULT $? 0 1 "${service} disable failed"
        systemctl enable "${service}"
        find ${symlink_file}
        CHECK_RESULT $? 0 0 "${service} enable failed"
    elif [ "${state}" == "disabled" ]; then
	symlink_file=$(systemctl enable "${service}" 2>&1 | grep "Created symlink" | awk '{print $3}' | tr -d '"')
        find ${symlink_file}
        CHECK_RESULT $? 0 0 "${service} enable failed"
        systemctl disable "${service}"
        find ${symlink_file}
        CHECK_RESULT $? 0 1 "${service} disable failed"
    elif [ "${state}" == "masked" ]; then
        LOG_INFO "Unit is masked, ignoring."
    elif [ "${state}" == "static" ]; then
        LOG_INFO "The unit files have no installation config,This means they are not meant to be enabled using systemctl."
    else
        LOG_INFO "Unit is indirect, ignoring."
    fi
}

function test_reload() {
    service=$1
    systemctl start "${service}"
    if systemctl cat "${service}" | grep -q "ExecReload"; then
        systemctl reload "${service}"
        CHECK_RESULT $? 0 0 "reload unit ${service} failed"
    else 
        systemctl reload "${service}" 2>&1 | grep "Job type reload is not applicable"
        CHECK_RESULT $? 0 0 "Job type reload is not applicable for unit ${service}"
    fi
    if ! systemctl status "${service}" | grep "Active: active"; then
        if systemctl status "${service}" | grep "inactive (dead)"; then
            systemctl status "${service}" | grep "Condition check" | grep "skip"
            CHECK_RESULT $? 0 0 "${service} reload causes the service status to change"
        else
            return 1
        fi
    fi
}
```

首先分析这一部分：

```
function test_execution() {
    service=$1
    log_time="$(date '+%Y-%m-%d %T')"
    test_restart "${service}"
    test_enabled "${service}"
    journalctl --since "${log_time}" -u "${service}" | grep -i "fail\|error" | grep -v -i "DEBUG\|INFO\|WARNING"
    CHECK_RESULT $? 0 1 "There is an error message for the log of ${service}"
}
```

1. 获取开始时间作为日志起点时间
2. 检查服务是否能够执行 restart 操作 (通过 test_restart 函数)
3. 检查服务是否可以执行 enable 操作 (通过 test_enabled 函数)
4. 检查服务日志中是否有错误信息 通过 grep 命令筛除掉一些不重要的信息
5. 用 CHECK_RESULT 检查 grep 的返回值

### 其他函数

* test_oneshot 只进行 enable 与 disable 测试
* test_reload 对服务进行 reload 测试

### 服务状态判定

```
systemctl restart "${service}"
CHECK_RESULT $? 0 0 "${service} restart failed"
SLEEP_WAIT 5
systemctl status "${service}" | grep "Active: active"
CHECK_RESULT $? 0 0 "${service} restart failed"
```

捕捉 status 的值 以此来进行判定

### 判定开机是否自启动

```
if [ "${state}" == "enabled" ]; then
symlink_file=$(systemctl disable "${service}" 2>&1 | grep "Removed" | awk '{print $2}' | awk '{print substr($0,1,length($0)-1)}' | tr -d '"')
    find ${symlink_file}
    CHECK_RESULT $? 0 1 "${service} disable failed"
    systemctl enable "${service}"
    find ${symlink_file}
    CHECK_RESULT $? 0 0 "${service} enable failed"
```

当 enable 一个服务的时候，systemd 会把这个服务文件创建一个链接到 /etc/systemd/system/multi-user.target.wants/

目录里（以这个目录为例，实际上还得看具体情况），直接去查找该文件是否存在即可。disable 同理，去查找，找不到就符合条件。

以上参考自赵老师的文档

### mugen 的超时时间

`TIMEOUT` 这个是超时时间，一旦超过这个时间，测试判定为超时并退出

备注：因为 一些 riscv 设备的性能不如 x86 设备 故一些测试用例可能会超时 因此可能需要手动修改 `TIMEOUT`

mugen 的计时方法：[参考赵老师文档的附录](https://github.com/openeuler-riscv/oerv-team/blob/main/cases/2024.12.25-mugen%E5%85%A5%E9%97%A8-%E8%B5%B5%E9%A3%9E%E6%89%AC.md#%E9%99%84%E5%BD%95%E4%B8%80-sleep_wait%E7%AD%89%E5%BE%85%E5%87%BD%E6%95%B0)

### OET_PATH 

它永远指向 mugen.sh 文件所在的目录

要引入一个脚本的话，可以直接从 `${OET_PATH}` 开始寻址

### 关于判定 python3

```
if python3 --version; then
    source "$OET_PATH/libs/locallibs/common_lib_python.sh"
else
    source "$OET_PATH/libs/locallibs/common_lib_shell.sh"
fi
```

如果当前环境里有 python3 则使用 common_lib_python.sh，否则就使用 shell 来实现同名函数

## mugen 如何执行测试套中的测试脚本

以下是 mugen.sh 这里我选取了部分进行分析学习

```
while getopts "c:af:r:dxb:s:m" option; do
    case $option in
```

这个 while 循环里有几种情况 其实就是判断你输入的参数

就是前文中提到的那几个

如果读取到命令参数 就对这个参数进行解析 执行 相应的操作

举个例子：

```
c)
        deploy_conf ${*//-c/} #把选项列表里除了-c的部分全部传递给deploy_conf
        ;;
```

如果读取到 c 就执行 deploy_conf

也就是进行配置操作

大概的运行流程就是：

1. while 循环读取参数
2. run_test_suite
3. run_test_case
4. exec_case
5. SLEEP_WAIT
6. 测试脚本
7. main 函数

备注： 2 和 3 看的是第一步循环里是否有读取到参数

之后 mugen 会寻找相应的 json 文件并检查测试环境是否符合条件

### exec_case 函数

```
function exec_case() {
    local cmd=$1
    local log_path=$2
    local case_name=$3
    local test_suite=$4

    exec 6>&1 #将当前的标准输出 (stdout) 复制到文件描述符 6
    exec 7>&2 #将当前的标准错误 (stderr) 复制到文件描述符 7
    exec >>"$log_path"/"$(date +%Y-%m-%d-%T)".log 2>&1 #把原本应该往终端的输出(stdout and stderr)重定向到日志文件,并且日志文件的名称正好就是运行该测试脚本的时间

    SLEEP_WAIT $TIMEOUT "$cmd"
    ret_code=$?

    exec 1>&6 6>&- #标准输出重定向回文件描述符 6（原始 stdout），并关闭文件描述符 6
    exec 2>&7 7>&- #标准错误重定向回文件描述符 7（原始 stdout），并关闭文件描述符 7

    test "$ret_code"x == "143"x && { #如果退出码是143则是超时
        cmd_pid=$(pgrep -f "$cmd") #给执行的命令相关的进程发送SIGKILL信号
        if [ -n "$cmd_pid" ]; then
            for pid in ${cmd_pid}; do
                pstree -p "$pid" | grep -o '([0-9]*)' | tr -d '()' | xargs kill -9
            done
        fi
        LOG_WARN "The case execution timeout." #往终端输出超时的信息
    }

    generate_result_file "$test_suite" "$case_name" "$ret_code"
}
```

以上注释摘自赵老师的文档

这里我不太懂 是用这个 exec_case 去判断是否超时吗？

* exec_case 之后就是开始执行测试了

# pcp 测试套执行

刚刚描述了现在我对 mugen 的基本认知 接下来会尝试再次去执行 pcp 测试套 包括测试环境的搭建与用例的执行

## 测试环境

### pcp 概述

PCP 全称是 Performance Co-Pilot

是一款功能强大的性能监控与分析框架，用于收集、记录、分析和可视化系统性能数据

它广泛用于 Linux 系统的性能调优、问题诊断和长期趋势分析

以上内容来源网络搜索

### 环境需要

Mugen 的 PCP 测试需要两台互通网络的测试机

感觉可以用 qemu 搭一下

备注：需要的工具 `dnf install -y qemu-system-riscv bridge-utils`

起了两台 qemu 使用 qemu 里面的 socket 进行通信

可以加两句在脚本里： `-netdev socket,id=net0,listen=:1234` 以及 `-netdev socket,id=net0,connect=127.0.0.1:1234`

大概还需要手动指定一下两台虚拟机的 mac 地址，保证是唯一的

大致流程：

1. 首先准备好镜像和启动脚本等必要文件
2. 在虚拟机内配置静态 ip 测试一下看看两台是不是都能互相 ping 通
3. 设置 ssh 免密登录 这样方便接下来的 pcp 测试进行
4. 在测试机中安装好 pcp
5. 配置 mugen 
6. `bash mugen.sh -f pcp`

### 实际测试

测试结果：

```
hu Apr 24 16:32:31 2025 - INFO  - A total of 74 use cases were executed, with 36 successes 38 failures and 0 skips.
```

成功用例：

```
[root@10 pcp]# cd succeed/
[root@10 succeed]# ls
oe_test_dbpmda               oe_test_pmconfig_pmie_check
oe_test_dstat_01             oe_test_pmdate
oe_test_dstat_02             oe_test_pmdumptext_001
oe_test_pcp2json_03          oe_test_pmdumptext_002
oe_test_pcp2json_04          oe_test_pmdumptext_003
oe_test_pcp2json_05          oe_test_pmjson
oe_test_pcp2xml_03           oe_test_pmlogger_daily_02
oe_test_pcp2xml_04           oe_test_pmlogger_daily_report
oe_test_pcp_atop_01          oe_test_pmrep_03
oe_test_pcp_collectl             oe_test_pmrep_04
oe_test_pcp_dmcache          oe_test_pmrep_05
oe_test_pcp_free             oe_test_pmrep_06
oe_test_pcp_pcp-import-collectl2pcp  oe_test_service_pcp-geolocate
oe_test_pcp_pcp-import-iostat2pcp    oe_test_service_pcp-reboot-init
oe_test_pcp_pcp-import-mrtg2pcp      oe_test_service_pmcd
oe_test_pcp_pcp-import-sar2pcp       oe_test_service_pmie
oe_test_pcp-pidstat_01           oe_test_service_pmlogger
oe_test_pcp-pmda-elasticsearch       oe_test_service_pmproxy
```

成功用例之一：

```
[root@10 pcp]# cd oe_test_pmdate/
[root@10 oe_test_pmdate]# ls
2025-04-24-00:29:25.log
[root@10 oe_test_pmdate]# cat 2025-04-24-00\:29\:25.log 
Python 3.11.11
Thu Apr 24 00:29:30 2025 - INFO  - Start to prepare the test environment.
Thu Apr 24 00:29:43 2025 - INFO  - pkgs:(pcp) is already installed
Thu Apr 24 00:30:06 2025 - INFO  - End to prepare the test environment.
Thu Apr 24 00:30:07 2025 - INFO  - Start to run test.
200424-00:30:07
250724-00:30:07
250419-00:30:07
250424-03:30:08
250424-00:25:08
250424-00:30:11
Thu Apr 24 00:30:09 2025 - INFO  - End to run test.
Thu Apr 24 00:30:09 2025 - INFO  - succeed to execute the case.
Thu Apr 24 00:30:10 2025 - INFO  - Start to restore the test environment.
Thu Apr 24 00:30:11 2025 - WARN  - no thing to do.
Thu Apr 24 00:30:12 2025 - INFO  - End to restore the test environment.
[root@10 oe_test_pmdate]# 
```

失败用例：

```
[root@10 failed]# ls
oe_test_dstat_03
oe_test_pcp
oe_test_pcp2json_01
oe_test_pcp2json_02
oe_test_pcp2xml_01
oe_test_pcp2xml_02
oe_test_pcp_atop_02
oe_test_pcp-iostat
oe_test_pcp-mpstat_pcp-numastat
oe_test_pcp_pcp-import-ganglia2pcp
oe_test_pcp-pidstat_02_pcp-ipcs
oe_test_pcp-summary_pcp-vmstat_pmcd_wait
oe_test_pcp-uptime_pcp-verify
oe_test_pmdumplog_01
oe_test_pmdumplog_02
oe_test_pmevent_01
oe_test_pmevent_02
oe_test_pmfind_pmgenmap_pmie2col_pminfo_01
oe_test_pmhostname_pmlock_pmlogger_check
oe_test_pminfo_02
oe_test_pminfo_03
oe_test_pmlogcheck_pmlogmv
oe_test_pmlogconf_pmlogsize
oe_test_pmlogger_daily_01
oe_test_pmlogger_merge_pmlogger_rewrite
oe_test_pmloglabel
oe_test_pmlogreduce_pmpause_pmpost_pmsleep
oe_test_pmlogsummary_01
oe_test_pmlogsummary_02
oe_test_pmprobe_01
oe_test_pmprobe_02
oe_test_pmpython_mkaf_pcp-python
oe_test_pmrep_01
oe_test_pmrep_02
oe_test_pmstat
oe_test_pmstore_install-sh
oe_test_pmval_01
oe_test_pmval_02
```

部分错误信息分析：

* oe_test_pcp.sh

```
[root@10 oe_test_pcp]# cat 2025-04-24-13\:16\:27.log 
Python 3.11.11
Thu Apr 24 13:16:32 2025 - INFO  - Start to prepare the test environment.
Thu Apr 24 13:16:43 2025 - INFO  - pkgs:(pcp) is already installed
Thu Apr 24 13:17:07 2025 - INFO  - End to prepare the test environment.
Thu Apr 24 13:17:08 2025 - INFO  - Start to run test.
pcp version 6.3.2
Cannot find a pcp-3min command to execute
Thu Apr 24 13:17:14 2025 - ERROR - oe_test_pcp.sh line 33
Thu Apr 24 13:17:17 2025 - ERROR - oe_test_pcp.sh line 35
Cannot find a pcp-@08 command to execute
Thu Apr 24 13:17:22 2025 - ERROR - oe_test_pcp.sh line 37
 hardware: 4 cpus, 17 disks, 1 node, 3659MB RAM
Thu Apr 24 13:17:31 2025 - ERROR - oe_test_pcp.sh line 41
Cannot find a pcp-/var/lib/pcp/pmns/root command to execute
Thu Apr 24 13:17:36 2025 - ERROR - oe_test_pcp.sh line 43
Cannot find a pcp-22 command to execute
Thu Apr 24 13:17:41 2025 - ERROR - oe_test_pcp.sh line 45
Cannot find a pcp-@08 command to execute
Please install pcp system tools package
 pmlogger: primary logger: /var/log/pcp/pmlogger/10.0.2.15/20250424.13.12
Thu Apr 24 13:17:53 2025 - ERROR - oe_test_pcp.sh line 51
Thu Apr 24 13:17:54 2025 - INFO  - End to run test.
Thu Apr 24 13:17:55 2025 - ERROR - failed to execute the case.
Thu Apr 24 13:17:55 2025 - INFO  - Start to restore the test environment.
Thu Apr 24 13:17:56 2025 - WARN  - no thing to do.
Thu Apr 24 13:17:57 2025 - INFO  - End to restore the test environment.
```

分析：

我觉得这个应该是测试脚本编写的有问题

其中 log 描述的错误里提到找不到命令

有的 pcp-xxx 的命令好像需要使用以下的命令才能得到输出

因此我认为这可能是导致错误的原因之一

我所知的常见的 pcp 工具包中的命令有：

* pmcd | 启动性能指标收集守护进程（Performance Metrics Collector Daemon）
* pminfo | 查看系统中可用的性能指标
* pmval | 获取某个性能指标的实时/历史值
* pmstat | 类似 vmstat，显示 CPU、内存、磁盘等摘要性能信息
* pmlogger | 记录性能数据到磁盘，用于离线分析
* pmlogsummary | 生成日志摘要报告（配合 pmlogger 使用）
* pmdumplog | 显示 .0 性能日志文件中的内容
* pmie | 事件引擎，可以做报警、自动化等
* pmproxy | 提供 REST API，供远程应用查询性能指标（适合 Web 前端）

原脚本：

```
#!/usr/bin/bash

# Copyright (c) 2021. Huawei Technologies Co.,Ltd.ALL rights reserved.
# This program is licensed under Mulan PSL v2.
# You can use it according to the terms and conditions of the Mulan PSL v2.
#          http://license.coscl.org.cn/MulanPSL2
# THIS PROGRAM IS PROVIDED ON AN "AS IS" BASIS, WITHOUT WARRANTIES OF ANY KIND,
# EITHER EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO NON-INFRINGEMENT,
# MERCHANTABILITY OR FIT FOR A PARTICULAR PURPOSE.
# See the Mulan PSL v2 for more details.
####################################
#@Author        :   zhujinlong
#@Contact       :   zhujinlong@163.com
#@Date          :   2020-10-14
#@License       :   Mulan PSL v2
#@Desc          :   pcp testing(pcp)
#####################################

source "common/common_pcp.sh"

function pre_test() {
    LOG_INFO "Start to prepare the test environment."
    deploy_env
    archive_data=$(pcp -h "$host_name" | grep 'primary logger:' | awk -F: '{print $NF}')
    LOG_INFO "End to prepare the test environment."
}

function run_test() {
    LOG_INFO "Start to run test."
    pcp --version | grep "$pcp_version"
    CHECK_RESULT $?
    pcp -a $archive_data -A 3min | grep 'Performance'
    CHECK_RESULT $?
    pcp -h $host_name | grep 'platform'
    CHECK_RESULT $?
    pcp -a $archive_data -O @08 -s 10 -t 2 | grep 'archive'
    CHECK_RESULT $?
    pcp -P | grep 'hardware'
    CHECK_RESULT $?
    pcp -a $archive_data -g | grep 'timezone'
    CHECK_RESULT $?
    pcp -a $archive_data -n /var/lib/pcp/pmns/root | grep 'services'
    CHECK_RESULT $?
    pcp -a $archive_data -p 22 | grep 'pmcd'
    CHECK_RESULT $?
    pcp -a $archive_data -S @08 -T @18 | grep "$archive_data"
    CHECK_RESULT $?
    pcp -Z Africa/Lagos | grep 'pmlogger'
    CHECK_RESULT $?
    pcp -a $archive_data -z | grep 'Performance Co-Pilot'
    CHECK_RESULT $?
    LOG_INFO "End to run test."
}

function post_test() {
    LOG_INFO "Start to restore the test environment."
    DNF_REMOVE
    LOG_INFO "End to restore the test environment."
}

main "$@"
```


# 总结

经过对赵老师的文档以及吴洁老师的视频内容

我现在对 mugen 测试框架有了更深入的了解与认识

在没有详细测试文档的情况下我也可以研究被测试的工具或软件本身

以编写测试用例和进行测试工作

关于 pcp 测试套，我起了两个 qemu，测试了一下是可以互相 ping 通的，并且 pmcd 服务监听的端口也转发到 node1 了，不知道为什么 pcp 远程连接的功能都不太能用，有可能是 pcp 工具在 riscv 平台上对于这一部分功能的支持还没有完善，也有可能是网络配置上还是出了问题

这周末我还会去试试配置运行其他的测试套 增进一下对 mugen 的掌握程度