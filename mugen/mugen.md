## mugen 的架构

* 测试用例： shell 脚本 存放在 testcases 下
* 测试套： 批量进行的一组测试用例
* 配置文件： 定义默认的一些参数
* 日志文件： log 文件 存放在 logs 下

## mugen 的运行流程

### 配置测试环境

* 运行 mugen.sh 脚本

```
有几个参数需要设置:

ip user password port

ip： 测试机的 ip 地址

user： 用户名

password： 密码

port： 测试机的 ssh 端口
```

* 执行测试套

1. 运行测试用例
2. 读取配置文件
3. 安装依赖
4. 收集日志保存到 log 文件
5. 判断结果 收集 exit code 

## pre_test() 函数

用途：

* 准备测试环境
* 检查环境是否满足要求
* 安装依赖

## run_test() 函数

用途：

* 执行测试用例
* 测试需要测试的内容
* 用 CHECK_RESULT 对比运行结果与预期结果

## post_test() 函数

用途：

* 清理恢复测试环境
* 删除临时文件
* 无论测试成功或失败都执行

## CHECK_RESULT 函数

用途：

* 验证运行结果是否符合预期

判断方法： $ + 上一命令的退出码或要进行对比的预期结果 + 失败时输出的错误信息

## admin_guide.json

这是一个测试配置文件


```
{
    "path": "$OET_PATH/testcases/doc-test/admin_guide",  测试用例目录
    "machine type": "kvm",  测试机类型
    "add network interface": 2,  给测试机添加网卡
    "add disk": [
        2,
        2,
        2,
        2
    ],  给测试机添加虚拟磁盘
    "machine num": 2,  需要测试机两台
    "cases": [  执行测试用例
        {
            "name": "oe_test_basic_del_user"  删除用户
        },
        {
            "name": "oe_test_basic_system_info"  查询系统信息
        },
        {
            "name": "oe_test_basic_set_pwd_empty"  设置空密码
        },
        {
            "name": "oe_test_httpd_check_configuration_status"  httpd 服务测试
        },
        {
            "name": "oe_test_lvm_pvdispay",  lvm 存储卷显示
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_basic_query_MEM"  内存信息查询
        },
        {
            "name": "oe_test_httpd_verify_web",  测试 http 请求访问网页
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_basic_Hwclcok_setting"  时钟显示与设定
        },
        {
            "name": "oe_test_basic_mod_current_user_pwd"  修改当前用户密码
        },
        {
            "name": "oe_test_lvm_lvreduce",  lvm 逻辑卷缩减
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_lvm_Manual_mount",  lvm 挂载卷
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_lvm_pvchange",  分配卷
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_lvm_lvremove_lvchange",  删除卷  修改卷属性
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_basic_local_setting"  语言地区设置
        },
        {
            "name": "oe_test_basic_query_cpu"  cpu 信息查询
        },
        {
            "name": "oe_test_lvm_lvcreate_lvdisplay",  创建卷 显示卷属性信息
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_lvm_vgextend",  拓展卷
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_basic_mod_user_home_directory"  修改用户 home 目录
        },
        {
            "name": "oe_test_lvm_auto_mount",  用 blkid 查询 uuid 并挂载
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_basic_mod_UID"  修改用户 uid
        },
        {
            "name": "oe_test_lvm_lvchange",  修改卷属性
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_lvm_lvresize_add",  修改卷大小 拓展或减小
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_basic_view_disk_info"  查询磁盘信息
        },
        {
            "name": "oe_test_dnf_package_operate"  安装 httpd
        },
        {
            "name": "oe_test_basic_configure_keyboard"  设置键盘布局
        },
        {
            "name": "oe_test_lvm_pvremvoe",  删除卷
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_lvm_vgcreate_vgdisplay",  创建卷 查看卷组
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_basic_config_Hostnamectl",  设置 hostname
            "machine num": 2
        },
        {
            "name": "oe_test_httpd_manage_module_ssl"  管理 ssl 模块
        },
        {
            "name": "oe_test_basic_set_time"  设置时间
        },
        {
            "name": "oe_test_basic_view_user_group_ids"  查看用户与组
        },
        {
            "name": "oe_test_dnf_source_set"  设置软件源
        },
        {
            "name": "oe_test_dnf_package_group_operate"  安装软件包组
        },
        {
            "name": "oe_test_lvm_shrink_roll_group",  缩减逻辑卷
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_basic_add_user"  添加用户
        },
        {
            "name": "oe_test_basic_related_account_info"  检查用户信息
        },
        {
            "name": "oe_test_lvm_lvresize_reduce",  减小逻辑卷大小
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_dnf_display_config_info"  显示当前的配置信息
        },
        {
            "name": "oe_test_lvm_lvextend",  逻辑卷扩容
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_basic_expiration_time_setting"  设置账户密码过期时间
        },
        {
            "name": "oe_test_lvm_vgchange_vgremove",  删除卷组
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_lvm_pvcreate",  创建物理卷
            "add disk": [
                2,
                2,
                2,
                2
            ]
        },
        {
            "name": "oe_test_mysql_backup_restore_db"  测试备份与恢复数据库
        },
        {
            "name": "oe_test_httpd_transfer_start_stop_restart"  启动重启 httpd 服务
        },
        {
            "name": "oe_test_mariadb_grant_delete_authorization"  用户权限 删除用户
        },
        {
            "name": "oe_test_mysql_configuration"  配置 mysql
        },
        {
            "name": "oe_test_mysql_create_view_user"  创建用户 查看用户
        },
        {
            "name": "oe_test_mysql_create_view_select_delete_db"  创建视图 mysql 基本操作
        },
        {
            "name": "oe_test_mariadb_install_run_uninstall"  安装 mariadb-server 卸载 运行
        },
        {
            "name": "oe_test_mariadb_create_view_select_delete_db"  mariadb 操作
        },
        {
            "name": "oe_test_mysql_grant_delete_authorization"  设置用户权限
        },
        {
            "name": "oe_test_mariadb_create_view_user" 创建视图与用户
        },
        {
            "name": "oe_test_mariadb_backup_restore_db" 备份数据库
        },
        {
            "name": "oe_test_httpd_verify_status"  查看 httpd 状态
        },
        {
            "name": "oe_test_mariadb_modify_delete_user"  删除用户
        },
        {
            "name": "oe_test_dnf_create_local_repository"  建立本地 repo
        },
        {
            "name": "oe_test_nmcli_config_dynamic_IP"  nmcli 设置 ip 连接
        },
        {
            "name": "oe_test_postgresql_create_delete_roles"  创建删除使用者
        },
        {
            "name": "oe_test_nginx_verify_web",  检测 web 服务是否成功搭建
            "machine num": 2  需要使用两台测试机
        },
        {
            "name": "oe_test_postgresql_backup_restore_db"  备份数据库
        },
        {
            "name": "oe_test_nmcli_query_link"  nmcli 查询连接状态
        },
        {
            "name": "oe_test_postgresql_create_run_delete"  安装运行卸载 postgresql
        },
        {
            "name": "oe_test_nmcli_mod_attribute"  添加配置网络连接
        },
        {
            "name": "oe_test_postgresql_change_roles" 修改使用者
        },
        {
            "name": "oe_test_postgresql_create_delete_db"  创建删除数据库
        },
        {
            "name": "oe_test_nmcli_config_static_IP"  nmcli 配置静态 ip 
        },
        {
            "name": "oe_test_nmcli_use_bonding"  配置 bonding 多网卡
        },
        {
            "name": "oe_test_nmcli_config_hostname"  配置主机 hostname
        },
        {
            "name": "oe_test_nginx_manage_status"  管理 nginx 并查看状态
        },
        {
            "name": "oe_test_mysql_modify_delete_user"  删除用户
        },
        {
            "name": "oe_test_nginx_check_configuration"  查看 nginx 的配置信息
        },
        {
            "name": "oe_test_procmgmt_who"  测试命令 who
        },
        {
            "name": "oe_test_procmgmt_ps_top_kill"  测试 ps top kill
        },
        {
            "name": "oe_test_set_XLL_Forwarding"  这个我不太懂用例是什么意思 可能还需要看看
        },
        {
            "name": "oe_test_procmgmt_run_programs_regularly"  将一批次的进程打印到屏幕上
        },
        {
            "name": "oe_test_vsftpd_start_stop"  启动 vsftpd 服务
        },
        {
            "name": "oe_test_server_verify_service_status"  查询服务状态
        },
        {
            "name": "oe_test_vsftpd_transfer_deleted",  删除 ftp 文件
            "machine num": 2
        },
        {
            "name": "oe_test_server_operate_service"  检查服务状态测试
        },
        {
            "name": "oe_test_procmgmt_create_another_cron_file"  创建 cron 文件
        },
        {
            "name": "oe_test_postgresql_grant_delete_authorization"  删除用户权限
        },
        {
            "name": "oe_test_vsftpd_transfer_download",  ftp 文件下载
            "machine num": 2
        },
        {
            "name": "oe_test_vsftpd_transfer_upload",  ftp 文件上传
            "machine num": 2 
        },
        {
            "name": "oe_test_vsftpd_conf"  配置 vsftpd 服务
        },
        {
            "name": "oe_test_ip_cmd_addr" ip addr 命令测试
        },
        {
            "name": "oe_test_ip_cmd_route"  ip route 命令测试
        },
        {
            "name": "oe_test_ip_cmd_rule"  ip rule 命令测试
        },
        {
            "name": "oe_test_ip_config_mtu"  配置设备 mtu
        },
        {
            "name": "oe_test_ip_config_multiple_address"  配置多个 ip 地址
        },
        {
            "name": "oe_test_ip_config_static_address"  设置 ip 地址
        },
        {
            "name": "oe_test_ip_config_static_route"  设置 ip route
        },
        {
            "name": "oe_test_basic_date_common"  date 命令测试
        }
    ]
}
```