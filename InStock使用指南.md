# InStock 股票系统使用指南

## 项目概述
InStock 是一个功能强大的股票量化分析系统，支持股票数据抓取、技术指标计算、K线形态识别、策略选股、回测验证和自动交易等功能。

## 一、环境准备

### 1. 安装 Python 3.11+
```bash
# 从官网下载安装：https://www.python.org/downloads/
# 配置国内镜像源
python -m pip config --global set global.index-url https://mirrors.aliyun.com/pypi/simple/
```

### 2. 安装 MySQL 数据库
```bash
# 从官网下载安装：https://dev.mysql.com/downloads/mysql/
# 记住设置的 root 密码
```

### 3. 安装 TA-Lib 库
```bash
# Windows: 下载预编译版本
# 访问：https://ta-lib.org/install/
# 按照官方说明安装对应系统版本
```

## 二、项目安装

### 1. 克隆项目
```bash
git clone https://github.com/myhhub/stock.git
cd instock
```

### 2. 安装依赖
```bash
python -m pip install -r requirements.txt
```

### 3. 配置数据库
编辑 `instock/lib/database.py`：
```python
db_host = "localhost"      # 数据库服务主机
db_user = "root"          # 数据库访问用户  
db_password = "your_password"  # 修改为你的数据库密码
db_port = 3306            # 数据库服务端口
db_database = "instockdb" # 数据库名称
```

## 三、可选配置

### 1. 配置代理（可选）
编辑 `instock/config/proxy.txt`：
```
# 格式：ip:port 或 username:password@ip:port
127.0.0.1:7860
52.13.248.29:3128
```

### 2. 配置东方财富 Cookie（推荐）
```bash
# 方式一：环境变量
setx EAST_MONEY_COOKIE "你的Cookie值"

# 方式二：文件配置
# 编辑 instock/config/eastmoney_cookie.txt
```

获取 Cookie 步骤：
1. 访问 https://quote.eastmoney.com/center/gridlist.html#hs_a_board
2. 登录账号（可选）
3. F12 打开开发者工具 → Network 标签
4. 刷新页面，找到请求头中的 Cookie 值

## 四、系统运行

### 1. 初始化和数据抓取
```bash
# Windows 用户直接运行
run_job.bat

# 或者手动执行 Python 命令
python execute_daily_job.py
```

支持的批量作业模式：
```bash
# 当前时间作业
python execute_daily_job.py

# 单个时间作业  
python execute_daily_job.py 2024-03-01

# 枚举时间作业
python execute_daily_job.py 2024-01-01,2024-02-08,2024-03-12

# 区间时间作业
python execute_daily_job.py 2024-01-01 2024-03-01
```

### 2. 启动 Web 服务
```bash
# Windows 用户
run_web.bat

# 或手动启动
python instock/web/web_service.py
```

访问：http://localhost:9988/

### 3. 启动交易服务（可选）
```bash
run_trade.bat
```

## 五、功能模块使用

### 1. 综合选股
- 支持 200+ 选股条件
- 包括基本面、技术面、消息面等维度
- 可自由组合选股条件

### 2. 技术指标计算
支持 32 种技术指标：
- MACD、KDJ、BOLL、RSI、CCI 等
- 结果与同花顺、通达信一致

### 3. K线形态识别
- 识别 61 种 K线形态
- 自动判断买入/卖出信号

### 4. 策略选股
内置策略包括：
- 放量上涨
- 均线多头
- 停机坪
- 回踩年线
- 突破平台
- 海龟交易法则等

### 5. 回测验证
- 验证策略成功率
- 支持历史数据回测

## 六、单独功能作业

```bash
# 综合选股作业
python selection_data_daily_job.py

# 基础数据实时作业（开盘后运行）
python basic_data_daily_job.py

# 基础数据非实时作业
python basic_data_other_daily_job.py

# 指标数据作业
python indicators_data_daily_job.py

# K线形态作业
python klinepattern_data_daily_job.py

# 策略数据作业
python strategy_data_daily_job.py

# 回测数据作业
python backtest_data_daily_job.py
```

## 七、Docker 部署（可选）

### 1. 安装数据库容器
```bash
docker network create InStockService

docker run -d --name InStockDbService \
    --network InStockService \
    -v /data/mariadb/data:/var/lib/instockdb \
    -e MYSQL_ROOT_PASSWORD=root \
    library/mariadb:latest
```

### 2. 安装系统容器
```bash
docker run -dit --name InStock --network=InStockService \
    -p 9988:9988 \
    -v /data/instockproxy.txt:/data/InStock/instock/config/proxy.txt \
    -v /data/eastmoneycookie.txt:/data/InStock/instock/config/eastmoney_cookie.txt \
    -e db_host=InStockDbService \
    mayanghua/instock:latest
```

## 八、使用建议

1. **首次运行**：建议先运行当前时间作业，获取最新数据
2. **定时任务**：将 `run_job.bat` 加入 Windows 任务计划，每个交易日 17:00 执行
3. **实时数据**：开盘期间可运行 `basic_data_daily_job.py` 获取实时数据
4. **历史数据**：使用区间时间作业补充历史数据
5. **数据管理**：建议安装 Navicat 等数据库管理工具

## 九、注意事项

- 股市有风险，投资需谨慎，系统仅供学习分析使用
- 东方财富 Cookie 需定期更新（建议每周）
- 交易功能涉及资金安全，请谨慎使用
- 系统运行日志保存在 `instock/log/` 目录下

## 十、主要功能详解

### 综合选股功能
综合选股支持股票范围、基本面、技术面、消息面、人气指标、行情数据等方面共200多个信息栏目进行自由组合选股。选股条件分为以下大类：

1. **股票范围**：市场、行业、地区、概念、风格、指数成份、上市时间
2. **基本面**：估值指标、每股指标、盈利能力、成长能力、资本结构与偿债能力、股本股东
3. **技术面**：MACD金叉、KDJ金叉、放量突破、低位资金净流入等
4. **消息面**：公告大事、机构关注情况、机构持股家数、机构持股比例
5. **人气指标**：股吧人气排名、人气排名变化等
6. **行情数据**：股价表现、成交情况、资金流向、行情统计、沪深股通

### 技术指标说明
系统基于 talib、pandas 计算指标，计算高效准确。调整个别指标公式，确保结果和同花顺、通信达结果一致。

支持指标包括：
1. MACD 2. KDJ 3. BOLL 4. TRIX，TRMA 5. CR 6. SMA 7. RSI 
8. VR，MAVR 9. ROC 10. DMI，+DI，-DI，DX，ADX，ADXR 11. W&R 
12. CCI 13. TR、ATR 14. DMA、AMA 15. OBV 16. SAR 17. PSY 
18. BRAR 19. EMV 20. BIAS 21. TEMA 22. MFI 23. VWMA
24. PPO 25. WT 26. Supertrend 27. DPO 28. VHF 29. RVI
30. FI 31. ENE 32. STOCHRSI

### K线形态识别
精准识别61种K线形态，支持用户自选形态识别。

识别形态包括：
1. 两只乌鸦 2. 三只乌鸦 3. 三内部上涨和下跌 4. 三线打击 5. 三外部上涨和下跌 
6. 南方三星 7. 三个白兵 8. 弃婴 9. 大敌当前 10. 捉腰带线 
11. 脱离 12. 收盘缺影线 13. 藏婴吞没 14. 反击线 15. 乌云压顶 
16. 十字 17. 十字星 18. 蜻蜓十字/T形十字 19. 吞噬模式 20. 十字暮星
... 等61种形态

形态识别结果：
- 负：出现卖出信号
- 0：没有出现该形态  
- 正：出现买入信号

### 筹码分布
筹码分布通过计算一定时间范围内股票的最高价、最低价、成交数，输出对应价格成交数占整个流通盘比值的分布图形。计算高效准确，结果与东方财富等专业软件一致，缺省计算210个交易日的成本，可以自行设定时间范围。

---

**特别声明**：股市有风险投资需谨慎，本系统只能用于学习、股票分析，投资盈亏概不负责。