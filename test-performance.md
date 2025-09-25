我需要完成Aurora 3.10版本和Graviton2/Graviton3/Graviton4实例的性能测试项目，并生成完整的分析报告。
## 环境配置
- 当前环境：macOS，已配置AWS CLI和Python3
- Python环境：source ~/Documents/VSCodeFolder/.venv/bin/activate
- 包管理：使用 uv pip install 安装Python包
- 密钥路径：~/Documents/VSCodeFolder/_aihome/kp-vgn.pem
- 测试区域：us-east-1
## 基础设施要求
- 使用现有VPC（如满足要求）
- 堡垒机：r6a.2xlarge，Amazon Linux 2023，开放22端口
- Aurora集群：3.10版本，3个实例分别为 db.r6g.2xlarge(主), db.r7g.2xlarge, db.r8g.2xlarge
- 安全组：不开放公网，但允许堡垒机访问Aurora（3306端口）
## 软件安装要求（关键改进）
- 堡垒机软件包：
* MySQL客户端：使用 mariadb105 (Amazon Linux 2023兼容)
* sysbench：从源码编译安装，需要先安装Development Tools和mariadb105-devel
* 编译依赖：git, automake, libtool, openssl-devel
- 安装顺序：先安装依赖包 → 编译sysbench → 验证安装
## 测试数据要求
- 使用sysbench oltp_read_write模板
- 8张表，每表200万记录
- 读写比例1:1
- 数据准备时间：约15-20分钟
## 测试执行要求
- 测试线程：32, 64, 128
- 每个测试：预热1分钟 + 测试2分钟 = 共3分钟
- **关键**：每个实例必须通过主备切换作为主节点进行测试（因为sysbench包含写操作）
- 测试顺序：r6g → r7g → r8g
- 切换等待时间：failover后等待60秒稳定，测试间隔30秒
## 定价数据（us-east-1，基于实时查询）
- 按需价格：db.r6g.2xlarge=$1.038/h, db.r7g.2xlarge=$1.106/h, db.r8g.2xlarge=$1.104/h
- 预留实例(1年)：db.r6g.2xlarge=$0.68/h, db.r7g.2xlarge=$0.851/h, db.r8g.2xlarge=$0.74/h
## 报告生成要求（重要更新）
需要生成3个不同类型的报告：
### 1. 性能测试报告
- 使用ECharts（中国国内CDN）进行图表渲染
- 商务风格HTML报告
- 包含：TPS/QPS/延迟P95对比、成本效益分析
- 文件名：aurora_graviton_performance_report.html
### 2. 架构分析报告
- 商务风格HTML报告，包含：
* Aurora版本兼容性详细分析（基于AWS CLI查询）
* 全球区域可用性和定价对比
* LTS版本发布时间线
* 技术规格对比
- 文件名：aurora_graviton_analysis_report.html
### 3. Draw.io架构图（重要新增）
- AWS官方风格架构图，使用标准AWS图标和颜色
- 包含VPC、多AZ、安全组、Aurora集群的完整架构
- 测试流程图，展示7个关键阶段和详细任务分解
- 文件格式：.drawio (可在diagrams.net中打开编辑)
- 文件名：aurora_graviton_architecture.drawio, aurora_graviton_test_process.drawio