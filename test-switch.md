我需要完成Aurora 3.10版本和Graviton2/Graviton4实例的切换测试项目，我会提供测试工具，你熟悉它，并完成测试，最终生成完整的分析报告。以下是测试的详细描述：

## 环境配置
- 当前环境：macOS，已配置AWS CLI和Python3
- Python环境：source ~/Documents/VSCodeFolder/.venv/bin/activate
- 包管理：使用 uv pip install 安装Python包
- 密钥路径：~/Documents/VSCodeFolder/_aihome/kp-vgn.pem
- 测试区域：us-east-1

## 基础设施要求
- 使用现有VPC（如满足要求）
- 堡垒机：r6a.2xlarge，Amazon Linux 2023，开放22端口
- Aurora集群：3.10版本，2个实例分别为 db.r6g.2xlarge(主),  db.r8g.2xlarge
- 安全组：允许堡垒机访问Aurora（3306端口）

## 软件安装要求（关键改进）
- 堡垒机软件包：
  * MySQL客户端：使用 mariadb105 (Amazon Linux 2023兼容)
  * sysbench：从源码编译安装，需要先安装Development Tools和mariadb105-devel
  * 编译依赖：git, automake, libtool, openssl-devel
- 安装顺序：先安装依赖包 → 编译sysbench → 验证安装
- 测试程序：~/Documents/VSCodeFolder/tanzhen目录下
    aurora_failover_monitor_3probes-v2.py - 包括MySQL读/写/DNS检测的3探针验证工具；
    aurora_config.json - 测试的配置信息，包括集群/用户名/密码/超时设置等；
    generate_html_report.py - HTML报告生成工具；
    README.md - 探针工具介绍
    aurora_probe_round2_20250918.jsonl - 之前的测试日志
    aurora_failover_report_round2.html - 之前的分析报告

## 测试执行要求
- 测试并发：使用测试程序模拟200个会话从堡垒机连接到Aurora集群
- 测试方法：在预热30秒之后，发起节点切换，从r6g切换到r8g，程序继续运行1分钟
- 测试评估：根据aurora_failover_monitor_3probes-v2.py的会话信息分析200个会话受影响的情况，包括受影响的时间点，影响的操作，恢复的时间，以及切换过程中会话的异常情况：中断/超时/重连等情况

## 报告输出
1.使用html输出报告，需要Chart渲染请使用中国区的echarts；
2.内容包括：实验设计，环境配置，操作时序图，会话随时间展示受影响的的情况，包括报错/超时/中断/恢复等情况；
3.给出切换对于高并发业务的影响模式描述，消除客户对业务影响的顾虑；