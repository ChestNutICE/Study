# **mugen pcp** 测试套分析

## 测试环境

宿主机： **Fedora 40 Server x86_64**

测试机: **openEuler 2503 riscv on QEMU**

## 操作步骤

* 配置 **mugen**: `bash mugen.sh -c --ip 100.0.2.15 --password openEuler12#$ --user root --port 12005`
* 运行测试套: `bash mugen.sh -f pcp`
* 等待测试用例运行结束，收集报错 **log** 进行分析

## 测试结果

**A total of 74 use cases were executed, with 33 successes 40 failures and 1 skips.**

## 收集到的错误信息

`testcase:oe_test_pcp_pcp-import-collectl2pcp`

`oe_test_pcp_pcp-import-ganglia2pcp`

`oe_test_pcp-mpstat_pcp-numastat`

`testcase:oe_test_pmrep_01`

`testcase:oe_test_pmlogcheck_pmlogmv`

`testcase:oe_test_service_pmproxy`

`testcase:oe_test_pmprobe_02`

`testcase:oe_test_pmrep_02`

`testcase:oe_test_pmevent_01`

`testcase:oe_test_pcp`

`testcase:oe_test_pmfind_pmgenmap_pmie2col_pminfo_01`

`testcase:oe_test_pmlogsummary_01`

`testcase:oe_test_pmstat`

`testcase:oe_test_service_pcp-geolocate`

`testcase:oe_test_pmval_02`

`testcase:oe_test_pmlogger_merge_pmlogger_rewrite`

`testcase:oe_test_pcp-uptime_pcp-verify`

`testcase:oe_test_pcp2xml_02`

`testcase:oe_test_pmloglabel`

`testcase:oe_test_pcp-pidstat_02_pcp-ipcs`

`testcase:oe_test_pmprobe_01`

`testcase:oe_test_pcp2json_02`

`testcase:oe_test_pmhostname_pmlock_pmlogger_check`

`testcase:oe_test_pmdumplog_01`

`testcase:oe_test_pcp2json_01`

`testcase:oe_test_pmlogsummary_02`

`testcase:oe_test_pcp_pcp-import-iostat2pcp`

`testcase:oe_test_pmdumplog_02`

`testcase:oe_test_pmstore_install-sh`

`testcase:oe_test_pmlogconf_pmlogsize`

`testcase:oe_test_service_pmlogger`

`testcase:oe_test_pcp2xml_01`

`testcase:oe_test_pmevent_02`

`testcase:oe_test_service_pmcd`

`testcase:oe_test_pmval_01`

`testcase:oe_test_pminfo_02`

`testcase:oe_test_service_pmie`

`testcase:oe_test_pmlogreduce_pmpause_pmpost_pmsleep`

`testcase:oe_test_pmpython_mkaf_pcp-python`

`testcase:oe_test_pcp-iostat`

`testcase:oe_test_pcp-summary_pcp-vmstat_pmcd_wait`

## 失败原因分析

`testcase:oe_test_pcp_pcp-import-collectl2pcp`:

ln: 无法创建符号链接 '/etc/init.d/collectl': 没有那个文件或目录

分析：可能测试系统使用 systemd 而不是 init.d

解决方法：重新配置 collectl

./INSTALL: 行 64: chkconfig: 未找到命令

分析：基于 systemd 的发行版可能默认不包含这个工具

解决方法：改用 systemctl 命令来管理服务

`oe_test_pcp_pcp-import-ganglia2pcp`

The test environment don't meet the requirements, please check your environment.

分析：应该是缺少依赖了

解决方法：安装依赖

`oe_test_pcp-mpstat_pcp-numastat`

分析：好像是代码有问题，错误信息：TypeError: 'str' object cannot be interpreted as ctypes.c_char_p

解决方法：修改完善脚本

`testcase:oe_test_pmrep_01`

pmrep: Connection refused; is pmcd running?

分析：缺少 pmrep 工具 pmcd 服务

解决方法：配置 pmrep 启动 pmcd 服务

`testcase:oe_test_pmlogcheck_pmlogmv`

pmlogcheck: no PCP archive files match ""

分析：缺少档案？（我不太清楚这个）

解决方法：想办法生成缺少的文件

`testcase:oe_test_service_pmproxy`

分析：配置文件不存在，

而且操作可能也有问题：

Failed to reload pmproxy.service: Job type reload is not applicable for unit pmproxy.service.

解决方法：补充配置文件，或许这个工具就缺失了

`testcase:oe_test_pmprobe_02`

分析：缺少文件

解决方法：补充缺少的文件

`testcase:oe_test_pmrep_02`

分析：缺少文件

解决方法：补充文件

`testcase:oe_test_pcp`

分析：测试系统内没有安装 pcp 工具集

解决方法：安装缺少的工具集

`testcase:oe_test_pmfind_pmgenmap_pmie2col_pminfo_01`

分析：缺少 pmcd 服务 文件缺少

`testcase:oe_test_pmlogsummary_01`

分析：文件缺失

解决方法：那个缺少的文件好像有好几个用例都是这个问题，不太清楚怎么解决

`testcase:oe_test_pmstat`

分析：可能还是因为无法连接那个 pmcd 服务导致的

解决方法：启用 pmcd 服务

`testcase:oe_test_service_pcp-geolocate`

分析：缺少 geolocate 服务

解决方法：安装并启用服务

`testcase:oe_test_pmval_02`

分析：这个似乎需要创建容器

解决方法：重新配置这个命令

`testcase:oe_test_pcp-uptime_pcp-verify`

分析：Connection refused ['192.168.2.10'] 没有成功连接

而且脚本应该也存在一些问题

解决方法：修改脚本，尝试重新进行连接 配置 pcp 服务

`testcase:oe_test_pcp2xml_02`

分析：缺少依赖

解决方法：安装依赖

`testcase:oe_test_pmloglabel`

分析：没有给使用参数

解决方法：重新配置使用参数

`testcase:oe_test_pcp-pidstat_02_pcp-ipcs`

分析：脚本问题

解决方法：重新修改脚本

`testcase:oe_test_pcp2json_02`

分析：依赖冲突

解决方法：删除冲突依赖

`testcase:oe_test_pmhostname_pmlock_pmlogger_check`

分析：脚本故障

解决方法：修改脚本

`testcase:oe_test_pmstore_install-sh`

分析：权限问题

解决方法：解决权限或者修改脚本

`testcase:oe_test_pmlogconf_pmlogsize`

分析：无法连接到 pmcd 服务

解决方法：配置启动 pmcd 服务

`testcase:oe_test_service_pmlogger`

分析：服务没配置好 或者没启用 也可能是没安装

解决方法：配置 pmlogger.service 并启用

`testcase:oe_test_service_pmcd`

分析：没安装这个服务 或者没配置好

解决方法：安装并配置 pmcd 服务

`testcase:oe_test_service_pmie`

分析：同上，服务没安装配置

解决方法：配置并安装服务

`testcase:oe_test_pmlogreduce_pmpause_pmpost_pmsleep`

分析：参数没给够

解决方法：重新进行配置和操作

`testcase:oe_test_pmpython_mkaf_pcp-python`

分析：缺少 pcp 工具集

解决方法：安装工具集

`testcase:oe_test_pcp-iostat`

分析：脚本问题

解决方法：修改脚本

`testcase:oe_test_pcp-summary_pcp-vmstat_pmcd_wait`

分析：缺少参数

解决方法：根据需要进行的操作与说明补充参数

## 报错原因总结

### 大概分为以下几个失败原因

* 使用的时候命令参数缺失了；解决方法：补充命令参数
* 未安装 pcp 工具集；解决方法：安装工具集
* 缺少依赖；解决方法：安装依赖
* 依赖冲突；解决方法：删除冲突的依赖
* init.d 与 systemd 的问题；解决方法：或许需要使用更新风格的方式管理与配置服务
* 脚本问题；解决方法：修改脚本
* 权限问题；解决方法：补充权限
* 缺少服务；解决方法：安装并配置服务
* 无法连接服务；解决方法：可能需要进行一些配置

## 我的学习进度与想法

### 目前来说 已经会运行已有的测试套和用例并收集 log 
### 大概可以找到报错的原因与问题
### 解决方法上有的我还不是很确定怎么弄 但是我觉得我是可以学会的
### 应该还是需要多参考已有的文章和指南
### 继续努力
