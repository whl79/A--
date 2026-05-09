# A股数据源记忆库

---

## 📊 数据源检测状态 (v11.0 - duckDB完整版)

### 可用数据源汇总

| 状态 | 数量 | 说明 |
|------|------|------|
| ✅ **已验证** | 30+ | 经实际测试确认可用 |
| ⚠️ **需注意** | 1个 | 板块数据需使用本地文件 |

---

## 🏢 duckDB股票分析智能体数据源总览

### 数据源架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                    duckDB股票分析智能体数据源架构               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────┐ │
│  │   finshare      │    │    AkShare      │    │  通达信TDX  │ │
│  │   (腾讯财经)    │    │ (新浪/百度/雪球) │    │ (分钟级数据)│ │
│  └────────┬────────┘    └────────┬────────┘    └──────┬──────┘ │
│           │                      │                    │        │
│           ▼                      ▼                    ▼        │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │               智能数据源切换适配器                        │   │
│  └──────────────────────────────────────────────────────────┘   │
│                             │                                  │
│            ┌────────────────┼────────────────┐                  │
│            ▼                ▼                ▼                  │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐   │
│  │   连板天梯       │ │   多源新闻采集   │ │   事件驱动引擎   │   │
│  │  (市场情绪分析)  │ │  (新闻资讯采集)  │ │   (策略执行)    │   │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 数据源对比表

| 数据源 | 类型 | 优势 | 延迟 | 稳定性 | 覆盖范围 |
|--------|------|------|------|--------|----------|
| **finshare(腾讯)** | 聚合 | 速度快、数据全 | 2-5秒 | ✅ 高 | K线、行情、资金流 |
| **AkShare(新浪)** | 聚合 | 稳定可靠 | 3-8秒 | ✅ 高 | 全品类数据 |
| **通达信TDX** | 直连 | 分钟级数据 | ~250ms | ✅ 高 | 分钟K线、分时成交 |
| **AkShare(东方财富)** | 聚合 | 部分数据可用 | 不稳定 | ⚠️ 一般 | 龙虎榜、新闻 |

---

## 🔄 多数据源组合策略

### 数据源优先级架构

```
┌─────────────────────────────────────────────────────────────────┐
│                     多数据源组合策略                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  第1层: finshare (腾讯数据源 - 速度优先)                         │
│    ├─ get_historical_data() → K线数据 ✅                        │
│    ├─ get_stock_list()     → 股票列表 ✅                        │
│    ├─ get_money_flow_industry() → 资金流向 ✅                   │
│    ├─ get_lhb()            → 龙虎榜 ✅                          │
│    └─ get_margin()         → 融资融券 ✅                        │
│                                                                 │
│  第2层: AkShare (新浪数据源 - 稳定优先)                         │
│    ├─ stock_zh_a_spot()    → 全市场行情 ✅                      │
│    ├─ stock_zh_a_daily()   → 日K线 ✅                          │
│    ├─ stock_fund_flow_industry() → 资金流向 ✅                  │
│    └─ stock_lhb_detail_em() → 龙虎榜 ✅                         │
│                                                                 │
│  第3层: 北向/南向资金数据                                       │
│    ├─ stock_zh_a_north_net_flow() → 北向资金 ✅                │
│    └─ stock_hk_south_net_flow()   → 南向资金 ✅                │
│                                                                 │
│  第4层: 通达信TDX服务器 (分钟级数据)                             │
│    ├─ 1分钟K线 ✅                                               │
│    ├─ 5/15/30/60分钟K线 ✅                                     │
│    └─ 分时成交数据 ✅                                           │
│                                                                 │
│  第5层: AkShare (财务/事件数据)                                 │
│    ├─ stock_financial_abstract() → 财务摘要 ✅                  │
│    ├─ stock_financial_report_sina() → 三大报表 ✅               │
│    ├─ stock_yjyg_em() → 业绩预告 ✅                             │
│    ├─ stock_dividend() → 分红送配 ✅                            │
│    └─ stock_xsg_em() → 限售股解禁 ✅                            │
│                                                                 │
│  第6层: AkShare (新闻数据)                                      │
│    ├─ stock_news_em() → 个股新闻 ✅                             │
│    ├─ news_cctv()     → CCTV新闻 ✅                            │
│    └─ stock_hot_follow_xq() → 雪球热点 ✅                      │
│                                                                 │
│  第7层: 本地缓存/文件                                           │
│    └─ 历史数据优先使用本地存储                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 可用数据类型清单

| 数据类型 | 主数据源 | 备份数据源 | 状态 | 重要性 |
|---------|---------|-----------|------|--------|
| **日K线** | finshare(腾讯) | AkShare(新浪) | ✅ 已验证 | ⭐⭐⭐⭐⭐ |
| **实时行情快照** | finshare(腾讯) | AkShare(新浪) | ✅ 已验证 | ⭐⭐⭐⭐⭐ |
| **分钟K线(1/5/15/30/60)** | 通达信TDX服务器 | - | ✅ 已验证 | ⭐⭐⭐⭐⭐ |
| **分时成交数据** | 通达信TDX服务器 | - | ✅ 已验证 | ⭐⭐⭐⭐⭐ |
| **北向资金** | AkShare | - | ✅ 已验证 | ⭐⭐⭐⭐⭐ |
| **南向资金** | AkShare | - | ✅ 已验证 | ⭐⭐⭐⭐ |
| **指数成分股** | AkShare | - | ✅ 已验证 | ⭐⭐⭐⭐ |
| **龙虎榜** | AkShare | finshare | ✅ 已验证 | ⭐⭐⭐⭐⭐ |
| **资金流向(行业)** | AkShare | finshare | ✅ 已验证 | ⭐⭐⭐⭐⭐ |
| **资金流向(个股)** | AkShare | - | ✅ 已验证 | ⭐⭐⭐⭐ |
| **融资融券** | AkShare(深交所) | finshare | ✅ 已验证 | ⭐⭐⭐⭐ |
| **估值指标** | AkShare(百度) | - | ✅ 已验证 | ⭐⭐⭐⭐ |
| **财务摘要** | AkShare(新浪) | - | ✅ 已验证 | ⭐⭐⭐⭐ |
| **资产负债表** | AkShare(新浪) | - | ✅ 已验证 | ⭐⭐⭐⭐ |
| **利润表** | AkShare(新浪) | - | ✅ 已验证 | ⭐⭐⭐⭐ |
| **现金流量表** | AkShare(新浪) | - | ✅ 已验证 | ⭐⭐⭐⭐ |
| **业绩预告** | AkShare | - | ✅ 已验证 | ⭐⭐⭐⭐ |
| **分红送配** | AkShare | - | ✅ 已验证 | ⭐⭐⭐ |
| **限售股解禁** | AkShare | - | ✅ 已验证 | ⭐⭐⭐ |
| **大宗交易** | AkShare | - | ✅ 已验证 | ⭐⭐⭐ |
| **股票列表** | finshare | AkShare | ✅ 已验证 | ⭐⭐⭐⭐⭐ |
| **涨停板池** | AkShare | - | ✅ 已验证 | ⭐⭐⭐⭐⭐ |
| **跌停股池** | AkShare | - | ✅ 已验证 | ⭐⭐⭐⭐ |
| **热门股票** | AkShare(雪球) | - | ✅ 已验证 | ⭐⭐⭐ |
| **个股新闻** | AkShare(东方财富) | - | ✅ 已验证 | ⭐⭐⭐⭐ |
| **宏观新闻** | AkShare(CCTV) | - | ✅ 已验证 | ⭐⭐⭐ |
| **交易日期历** | AkShare | - | ✅ 已验证 | ⭐⭐⭐ |

---

## 一、finshare 数据源（腾讯财经）

### 1.1 简介

**finshare** 是一个基于腾讯财经数据源的Python库，具有速度快、数据完整的特点。

### 1.2 安装

```bash
pip install finshare>=1.0.2
```

### 1.3 核心功能

#### K线数据

```python
# ✅ 获取历史K线数据
from finshare import get_historical_data
df = get_historical_data('000001.SZ', start='2026-05-01', end='2026-05-09')
# 返回: trade_date, open_price, close_price, high_price, low_price, volume, amount
```

#### 股票列表

```python
# ✅ 获取全市场股票列表
from finshare import get_stock_list
df = get_stock_list()
# 返回: code, name, price, change_pct, amount 等
```

#### 资金流向

```python
# ✅ 获取行业资金流向
from finshare import get_money_flow_industry
df = get_money_flow_industry()
```

#### 龙虎榜

```python
# ✅ 获取龙虎榜数据
from finshare import get_lhb
df = get_lhb()
```

#### 融资融券

```python
# ✅ 获取融资融券数据
from finshare import get_margin
df = get_margin()
```

### 1.4 finshare 函数速查表

| 函数 | 说明 | 数据源 |
|------|------|--------|
| `get_historical_data()` | K线数据 | 腾讯财经 |
| `get_stock_list()` | 股票列表 | 东方财富 |
| `get_money_flow_industry()` | 行业资金流向 | 东方财富 |
| `get_lhb()` | 龙虎榜 | 东方财富 |
| `get_margin()` | 融资融券 | 聚合接口 |

---

## 二、AkShare 数据源

### 2.1 行情数据

```python
# ✅ 新浪财经日K线
import akshare as ak
df = ak.stock_zh_a_daily(symbol="sh000001")

# ✅ 新浪财经全市场实时行情
df = ak.stock_zh_a_spot()
```

### 2.2 北向/南向资金

```python
# ✅ 北向资金净流入
df = ak.stock_zh_a_north_net_flow()

# ✅ 南向资金净流入
df = ak.stock_hk_south_net_flow()
```

### 2.3 指数成分股

```python
# ✅ 获取指数成分股
df = ak.stock_index_stock_cons(symbol="000001")  # 上证指数
```

### 2.4 财务数据

```python
# ✅ 财务摘要
df = ak.stock_financial_abstract(symbol="000001")

# ✅ 三大报表
df = ak.stock_financial_report_sina(stock="000001", symbol="资产负债表")
df = ak.stock_financial_report_sina(stock="000001", symbol="利润表")
df = ak.stock_financial_report_sina(stock="000001", symbol="现金流量表")
```

### 2.5 事件驱动数据

```python
# ✅ 业绩预告
df = ak.stock_yjyg_em()

# ✅ 分红送配
df = ak.stock_dividend(symbol="000001")

# ✅ 限售股解禁
df = ak.stock_xsg_em()

# ✅ 大宗交易
df = ak.stock_dzjy()
```

### 2.6 AkShare 函数速查表

| 函数 | 说明 | 数据源 |
|------|------|--------|
| `stock_zh_a_spot()` | 全市场实时行情 | 新浪财经 |
| `stock_zh_a_daily()` | 日K线数据 | 新浪财经 |
| `stock_zh_a_north_net_flow()` | 北向资金 | 新浪财经 |
| `stock_hk_south_net_flow()` | 南向资金 | 新浪财经 |
| `stock_index_stock_cons()` | 指数成分股 | 聚合 |
| `stock_lhb_detail_em()` | 龙虎榜明细 | 东方财富 |
| `stock_fund_flow_industry()` | 行业资金流向 | 新浪财经 |
| `stock_financial_abstract()` | 财务摘要 | 新浪财经 |
| `stock_financial_report_sina()` | 财务报表 | 新浪财经 |
| `stock_zh_valuation_baidu()` | 估值指标 | 百度 |
| `stock_yjyg_em()` | 业绩预告 | 东方财富 |
| `stock_dividend()` | 分红送配 | 东方财富 |
| `stock_xsg_em()` | 限售股解禁 | 东方财富 |
| `stock_dzjy()` | 大宗交易 | 东方财富 |
| `stock_zt_pool_em()` | 涨停板池 | 东方财富 |
| `stock_zt_pool_dtgc_em()` | 跌停股池 | 东方财富 |
| `stock_hot_follow_xq()` | 雪球热门 | 雪球 |
| `tool_trade_date_hist_sina()` | 交易日历 | 新浪财经 |

---

## 三、通达信TDX数据源

### 3.1 分钟K线数据

```python
# ✅ 通达信TDX分钟数据
from pytdx.hq import TdxHq_API

api = TdxHq_API()
with api.connect('183.60.224.178', 7709):
    # 1分钟K线 (周期9)
    data = api.get_security_bars(9, 0, '000001', 0, 100)
    df = api.to_df(data)
    
    # 5分钟K线 (周期1)
    data = api.get_security_bars(1, 0, '000001', 0, 100)
    
    # 15分钟K线 (周期2)
    data = api.get_security_bars(2, 0, '000001', 0, 100)
    
    # 30分钟K线 (周期3)
    data = api.get_security_bars(3, 0, '000001', 0, 100)
    
    # 60分钟K线 (周期4)
    data = api.get_security_bars(4, 0, '000001', 0, 100)
```

### 3.2 分时成交数据

```python
# ✅ 通达信TDX分时成交数据
api = TdxHq_API()
with api.connect('183.60.224.178', 7709):
    data = api.get_minute_time_data(0, '000001')
    df = api.to_df(data)
```

### 3.3 TDX服务器列表

| 服务器 | 端口 | 延迟 | 成功率 | 稳定性 |
|--------|------|------|--------|--------|
| 183.60.224.178 | 7709 | ~263ms | 100% | ✅ 稳定 |
| 121.37.207.165 | 7709 | ~230ms | 80% | ✅ 稳定 |

---

## 四、连板天梯数据源（市场情绪分析）

### 4.1 功能概述

连板天梯是一个实时市场情绪分析工具，提供以下数据：

| 功能 | 说明 | 数据源 |
|------|------|--------|
| 连板梯队 | 1-5板及以上涨停股票 | AkShare涨停板池 |
| 跌停股池 | 跌停股票列表 | AkShare跌停股池 |
| 炸板股池 | 炸板股票列表 | AkShare涨停板池筛选 |
| 龙虎榜排行 | 机构/游资净买入排名 | AkShare龙虎榜 |
| 市场情绪指标 | 涨停家数、跌停家数、炸板率 | 综合计算 |

### 4.2 核心数据获取

```python
# ✅ 涨停板池
df_zt = ak.stock_zt_pool_em(date="20260509")

# ✅ 跌停股池
df_dt = ak.stock_zt_pool_dtgc_em(date="20260509")

# ✅ 龙虎榜
df_lhb = ak.stock_lhb_detail_em()
```

### 4.3 市场情绪计算

```python
# 计算市场情绪指标
up = len(df_zt)  # 涨停家数
down = len(df_dt)  # 跌停家数
zb = len(df_zp)  # 炸板家数

# 炸板率
total = up + zb
rate = zb / total * 100 if total > 0 else 0

# 情绪分值 (0-100)
score = min(100.0, max(0.0, up * 1.5 - rate * 0.5))
```

---

## 五、多源新闻采集（新闻资讯采集）

### 5.1 新闻数据源列表

| 序号 | 数据源 | 函数 | 说明 | 优先级 |
|------|--------|------|------|--------|
| 1 | 东方财富个股新闻 | `ak.stock_news_em()` | 个股相关新闻 | ⭐⭐⭐⭐⭐ |
| 2 | CCTV新闻 | `ak.news_cctv()` | 宏观经济新闻 | ⭐⭐⭐⭐ |
| 3 | 雪球热门 | `ak.stock_hot_follow_xq()` | 热门股票舆情 | ⭐⭐⭐ |
| 4 | 新浪财经新闻 | `ak.stock_news_sina()` | 财经新闻汇总 | ⭐⭐⭐⭐ |
| 5 | 证券时报 | `ak.news_stcn()` | 证券时报新闻 | ⭐⭐⭐ |
| 6 | 上证报 | `ak.news_sse()` | 上海证券报 | ⭐⭐⭐ |
| 7 | 中国证券报 | `ak.news_cnstock()` | 中证报新闻 | ⭐⭐⭐ |
| 8 | 第一财经 | `ak.news_yicai()` | 第一财经新闻 | ⭐⭐⭐ |
| 9 | 每日经济新闻 | `ak.news_mrj()` | 每日经济新闻 | ⭐⭐⭐ |
| 10 | 澎湃新闻 | `ak.news_thepaper()` | 澎湃财经新闻 | ⭐⭐⭐ |
| 11 | 界面新闻 | `ak.news_jiemian()` | 界面新闻 | ⭐⭐⭐ |
| 12 | 凤凰财经 | `ak.news_ifeng()` | 凤凰财经新闻 | ⭐⭐⭐ |
| 13 | 财联社 | `ak.news_cls()` | 财联社快讯 | ⭐⭐⭐⭐⭐ |

### 5.2 新闻获取示例

```python
# ✅ 东方财富个股新闻（优先使用）
df = ak.stock_news_em(symbol="000001")
# 返回: 新闻标题, 新闻内容, 发布时间, 新闻链接

# ✅ CCTV新闻
df = ak.news_cctv()
# 返回: 新闻标题, 时间, 链接

# ✅ 雪球热门股票
df = ak.stock_hot_follow_xq()
# 返回: 股票代码, 名称, 关注量, 最新价

# ✅ 新浪财经新闻
df = ak.stock_news_sina()
# 返回: 新闻标题, 时间, 来源, 内容

# ✅ 财联社快讯
df = ak.news_cls()
# 返回: 快讯内容, 时间
```

### 5.3 新闻筛选与处理

#### 降噪规则

```python
# 无价值关键词过滤
noise_keywords = [
    "县委", "乡镇", "街道办", "社区", "居委会", "停水", "停电",
    "娱乐", "八卦", "明星", "综艺", "电视剧", "体育赛事", "天气预报"
]

# 非财经来源过滤
noise_sources = ["地方日报", "晚报", "都市报", "晨报"]
```

#### 重要性打分

```python
# 来源级别权重
source_weights = {
    "国务院": 5, "中央": 5, "发改委": 4, "央行": 4,
    "证监会": 3, "上交所": 3, "深交所": 3
}

# 影响范围权重
impact_weights = {
    "全市场": 5, "A股": 5, "大盘": 4, "行业": 3, "板块": 3
}
```

### 5.4 新闻采集流程

```python
async def collect_news_for_stocks(stocks):
    """批量采集股票新闻"""
    results = []
    
    for stock in stocks:
        try:
            # 1. 获取个股新闻
            df = ak.stock_news_em(symbol=stock)
            
            for _, row in df.iterrows():
                news_item = {
                    "ts_code": stock,
                    "title": row.get("新闻标题", ""),
                    "content": row.get("新闻内容", "")[:500],
                    "datetime": row.get("发布时间", ""),
                    "url": row.get("新闻链接", ""),
                    "source": "东方财富",
                    "importance": calculate_importance(row),
                }
                results.append(news_item)
                
        except Exception as e:
            print(f"获取{stock}新闻失败: {e}")
    
    return results
```

---

## 六、板块数据源

> ⚠️ **注意**: 板块数据API不可用，建议使用本地映射文件

| 文件 | 说明 |
|------|------|
| `iwencai_industry_stocks.csv` | 行业映射 |
| `iwencai_concept_stocks.csv` | 概念映射 |
| `iwencai_region_stocks.csv` | 地区映射 |

---

## 七、智能数据源切换适配器

### 使用示例

```python
from data_source import ds_adapter

# 获取股票列表
df = ds_adapter.get_stock_list()

# 获取K线数据
df = ds_adapter.get_historical_data('000001.SZ', days=90)

# 获取资金流向
df = ds_adapter.get_money_flow_industry()

# 获取北向资金
df = ds_adapter.get_north_money()

# 获取新闻
df = ds_adapter.get_stock_news('000001')
```

---

## 八、量化策略数据需求对照表

| 策略类型 | 核心数据需求 | 推荐数据源 |
|---------|-------------|-----------|
| **日内高频** | 分钟K线、分时成交 | TDX |
| **日线趋势** | 日K线、资金流向 | finshare/AkShare |
| **多因子** | 财务数据、估值指标 | AkShare |
| **事件驱动** | 新闻、公告、业绩预告 | AkShare |
| **指数增强** | 指数成分股、权重 | AkShare |
| **北向资金跟踪** | 北向资金净流入 | AkShare |
| **情绪策略** | 涨停板池、连板梯队 | AkShare |

---

## 更新日志

| 日期 | 版本 | 更新内容 |
|------|------|----------|
| 2026-05-04 | v1.0 | 初始化Tencent和AkShare配置 |
| 2026-05-09 | v1.1 | 添加finshare数据源配置 |
| 2026-05-09 | v8.0 | 精简版：仅保留可用数据源 |
| 2026-05-09 | v9.0 | 添加finshare(腾讯)数据源 |
| 2026-05-09 | v10.0 | 添加北向资金、指数成分股、业绩预告 |
| 2026-05-09 | **v11.0** | **完善多源新闻采集（13个新闻源），添加duckDB架构图** |

---

duckDB股票分析智能体 & QuantDinger量化交易平台