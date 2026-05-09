# A-股数据源记忆库 · 量化投资者的数据百科全书
30+ 已验证数据接口 | 多源自动切换 | 分钟级行情 | 13大新闻源 | 连板天梯情绪分析
本仓库是一份 A股量化数据源完整清单，涵盖了从日K线、实时行情、分钟数据、资金流向、财务数据到新闻采集、连板情绪等所有核心数据接口。基于 finshare + AkShare + 通达信TDX 构建，提供智能数据源优先级切换策略，让您写策略时不再为“数据从哪里来”而烦恼。

✨ 核心亮点
✅ 30+ 已验证数据源（全部实测可用，非理论列表）
⚡ 多源智能切换：finshare（腾讯）速度优先 → AkShare（新浪）稳定兜底 → TDX 分钟级补充
📊 分钟级数据支持：1/5/15/30/60分钟K线 + 分时成交（通达信直连）
📰 13大新闻源采集：东方财富、财联社、CCTV、雪球、证券时报… 带降噪与重要性打分
🧠 市场情绪分析：连板天梯、炸板率、龙虎榜排行、情绪分值计算
📁 板块映射文件：行业/概念/地区本地映射（解决板块API不可用问题）
🐍 开箱即用的代码示例：每个接口都配有 Python 调用示例

📦 数据源覆盖一览
数据类型	主数据源	状态
日K线 / 实时行情	finshare(腾讯) / AkShare(新浪)	✅
分钟K线(1/5/15/30/60)	通达信TDX	✅
北向/南向资金	AkShare	✅
龙虎榜	AkShare / finshare	✅
行业/个股资金流向	AkShare / finshare	✅
融资融券	AkShare / finshare	✅
财务三大报表 + 摘要	AkShare(新浪)	✅
业绩预告 / 分红 / 解禁 / 大宗交易	AkShare(东方财富)	✅
涨停板池 / 跌停股池 / 炸板股池	AkShare	✅
个股新闻 + 宏观新闻	13个新闻源聚合	✅

🚀 快速开始
# 获取日K线（自动选择最优数据源）
from finshare import get_historical_data
df = get_historical_data('000001.SZ', start='2026-05-01', end='2026-05-09')

# 获取全市场实时行情
import akshare as ak
df = ak.stock_zh_a_spot()

# 获取1分钟K线（通达信）
from pytdx.hq import TdxHq_API
api = TdxHq_API()
with api.connect('183.60.224.178', 7709):
    data = api.get_security_bars(9, 0, '000001', 0, 100)

# 采集个股新闻
df = ak.stock_news_em(symbol="000001")

详细接口清单、参数说明、异常处理方案请查阅仓库内的 README.md。

🧭 适用场景
量化策略开发（日线/分钟级/事件驱动/多因子）
市场情绪监控（连板天梯、炸板率）
新闻舆情分析（13个来源自动采集）
数据源替换与降级方案设计
学习 A股数据接口的完整知识库

📁 仓库结构
.
├── A股数据源记忆库.md      # 主文档（全部数据源说明+代码示例）
├── iwencai_industry_stocks.csv   # 行业映射文件
├── iwencai_concept_stocks.csv    # 概念映射文件
└── iwencai_region_stocks.csv     # 地区映射文件

⚠️ 板块数据因API不可用，请使用本地映射文件。

🤝 贡献与反馈
如果您发现某个数据源失效，或希望添加更多接口，欢迎提 Issue 或 PR。
本仓库将持续跟随 A股数据源变化而更新。

📄 许可证
MIT License — 可自由使用、修改、分享，保留版权声明即可。

⭐ 如果这个仓库对您有帮助，请点亮 Star，让更多量化爱好者看到。
或电联 y20369@qq.com
