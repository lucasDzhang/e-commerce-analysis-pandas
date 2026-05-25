# 在线零售数据分析项目

## 1 项目背景

本项目针对Kaggle上的online-retail-Ⅱ和olist数据集进行深度分析，旨在通过数据驱动的方式理解业务增长模式，识别关键驱动因素，并为运营决策提供数据支持。

**核心分析目标**：
- 拆解 GMV（商品交易总额）周期波动的来源
- 识别用户价值分层和结构变化
- 量化运营策略（如优惠券活动）的效果

## 2 数据来源

### 2.1 主要数据集
- **Online Retail II**（UCI Machine Learning Repository）
  - 时间跨度：2009-12-01 至 2011-12-09
  - 数据量：106 万 + 条交易记录
  - 覆盖用户：约 82 万条客户 ID 记录
  - 地理位置：主要覆盖英国市场

### 2.2 辅助数据集
- **Olist Brazilian E-commerce Dataset**
  - 用于客户行为漏斗分析和 A/B 测试验证

## 3 数据处理流程

```
raw data → clean → analysis → dashboard
```

### 3.1 数据清洗 pipeline
1. **去重处理**：删除重复交易记录
2. **缺失值处理**：删除 Customer ID 缺失的订单（无法归因）
3. **异常值过滤**：剔除 Quantity ≤ 0 或 Price ≤ 0 的退款/无效交易
4. **时间格式标准化**：转换为 datetime 格式
5. **核心指标构造**：amount = Quantity × Price

### 3.2 分析方法论
- **GMV 拆解框架**：GMV = 用户数 × 人均订单数 (APO) × 客单价 (AOV)
- **Cohort 分析**：追踪同组用户的行为变化
- **RFM 用户分层**：基于频率、金额、最近一次消费时间进行用户价值分层
- **A/B 测试**：优惠券效果评估（多重检验校正 + 置信区间 + 效应量）

## 4 使用工具

| 工具类别 | 技术栈 |
|---------|--------|
| **编程语言** | Python 3.8+ |
| **数据处理** | Pandas, NumPy |
| **数据可视化** | Matplotlib, Seaborn, Plotly |
| **统计分析** | SciPy |
| **开发环境** | Jupyter Notebook |
| **版本控制** | Git |

## 5 项目结构

```
purePythonProject/
├── notebooks/                    # Jupyter Notebook 分析脚本
│   ├── 01_analysis.ipynb        # 探索性数据分析 (EDA)
│   ├── 02_GMV_analysis.ipynb    # GMV 驱动因素拆解
│   ├── 03_Cohort_analysis.ipynb # 用户 Cohort 分析
│   ├── 04_User_behavior_analysis.ipynb  # 用户行为漏斗
│   ├── 05_AB_test_coupon_analysis.ipynb # A/B 测试分析
│   └── analysis.py              # 核心分析函数库
├── outputs/                      # 分析报告输出
│   └── report.md                # 详细分析报告
├── README.md                     # 项目说明文档
└── requirements.txt              # Python 依赖包
```

## 6 核心分析模块

### 6.1 探索性数据分析 (01_analysis.ipynb)
- 数据质量评估（缺失值、异常值检测）
- 描述性统计分析
- 数据分布可视化

### 6.2 GMV 驱动因素分析 (02_GMV_analysis.ipynb)
- **结构性拆解**：将 GMV 拆解为用户数、APO、AOV 三个维度
- **相关性分析**：量化各因素对 GMV 的贡献度
- **周期性识别**：发现 9-11 月为销售高峰期

**关键发现**：
- GMV 与用户数相关系数达 **0.96**（高度相关）
- GMV 与人均订单相关系数为 **0.53**（中等相关）
- GMV 与客单价相关系数仅 **0.04**（几乎无关）
- **结论**：GMV 主要由用户规模扩张驱动，属于"走量型增长"模式

### 6.3 用户 Cohort 分析 (03_Cohort_analysis.ipynb)
- 定义新用户：订单初次出现的月份为用户的 Cohort 月份
- 追踪同组用户的留存和消费行为变化
- **发现**：老用户是 GMV 的基本盘，新用户贡献波动较弱

### 6.4 用户行为分层 (04_User_behavior_analysis.ipynb)
- **RFM 模型应用**：
  - 按购买频率分位数（P50, P80）划分高/中/低频用户
  - 固定标签后追踪各群体后续表现
- **关键发现**：
  - Top 10% 老用户贡献 44%-64% 的 GMV
  - 高频用户是主力（贡献 50%-70%）
  - 用户结构存在不稳定性，头部用户占比波动大

### 6.5 A/B 测试分析 (05_AB_test_coupon_analysis.ipynb)
- **实验设计**：
  - 基于 MD5 哈希值进行随机分组（对照组 vs 实验组）
  - SRM 检验：p=0.35，分流比例正常（1:1）
  - 模拟优惠券对不同 RFM 人群的差异化 uplift 效果
- **统计方法**：
  - 整体检验：双样本 t 检验（Welch's t-test）
  - 分层检验：按 RFM 用户类型进行亚组分析
  - 多重检验校正：Bonferroni 校正（α'=0.05/7≈0.0071）
  - 效应量：Cohen's d + 95% 置信区间
- **核心发现**：
  - 全量发券效果不显著（p=0.099）
  - **New Customers**（新客）：GMV 提升 **11.93%**（p=0.0028✅）
  - **Promising**（一般保持用户）：GMV 提升 **17.62%**（p=0.0001✅）
  - 其他人群效果不显著，不建议全量投放

## 7 业务洞察与建议

### 7.1 增长模式识别
- **模式**：典型的"走量型增长结构"
- **驱动因素**：用户规模 + 购买频次
- **风险点**：对头部用户依赖度高，结构不稳定

### 7.2 运营建议

1. **用户运营**：
   - 重点维护高频老用户，建立会员体系
   - 发展中腰部用户，降低对头部用户的依赖

2. **促销策略**：
   - 在 9-11 月销售高峰期加大营销投入
   - 针对高频用户设计专属活动

3. **优惠券精准投放**（基于 A/B 测试）：
   - ✅ **重点投放**：New Customers（新客）、Promising（一般保持用户）
   - ❌ **停止投放**：Champions、Loyal Customers 等高价值用户（自然复购率高，无需补贴）
   - 策略：从全量发券转为精准定向，提升 ROI

4. **定价策略**：
   - 价格敏感度低，可适度优化定价策略
   - 避免过度依赖折扣促销

5. **风险控制**：
   - 监控头部用户流失风险
   - 建立用户价值预警机制

## 8 安装与运行

### 8.1 环境配置
```bash
# 创建虚拟环境（可选）
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 安装依赖
pip install -r requirements.txt
```

### 8.2 运行分析
```bash
# 进入 notebooks 目录
cd notebooks

# 打开 Jupyter Notebook
jupyter notebook

# 按顺序运行 Notebook：
# 1. 01_analysis.ipynb
# 2. 02_GMV_analysis.ipynb
# 3. 03_Cohort_analysis.ipynb
# 4. 04_User_behavior_analysis.ipynb
# 5. 05_AB_test_coupon_analysis.ipynb
```

## 9 参考资源

- [UCI Machine Learning Repository - Online Retail II](https://archive.ics.uci.edu/ml/datasets/online+retail+ii)
- [Olist E-commerce Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

---

**作者**：Lucas   
**创建时间**：2026-05-02  
**最后更新**：2026-05-02
