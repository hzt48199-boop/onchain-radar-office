# 链上雷达办公室：教程与接入说明

给 Muse 或任何开发工具用。目标：把 `index.html` 小样升级成接真实数据的版本。

## 1. 现在有什么

`index.html` 是单文件，无需构建，双击即可打开。

- 顶部：8 个智能体状态卡（CHIEF、SCAN、VET、WALLET、QUANT、SIZE、RISK、CRAWLER）
- 一级：PUMP / FOMO / GMGN 三个可点击卡片，点击后办公室、新币榜、钱包追踪都切到该来源
- 中间：Canvas 办公室动画。8 只蜘蛛从各数据口抓数据包，送到中央 HUB，再弹到墙上大屏
- 下方：一级新币榜（可点击一行派蜘蛛审查）、热点钱包追踪、二级主流币和热门币
- 底部滚动条：二级币价
- 目前全部是本地随机模拟数据，页面上已标注“模拟数据”

## 2. 推荐架构

```
数据源 → 后端（Node/Python，持有 API key）→ WebSocket → 前端 index.html
```

- API key 和钱包私钥只放后端，不要写进前端。
- 前端只接收整理好的 JSON，再调用页面暴露的 `window.OfficeFeed`。

## 3. 前端接入口：window.OfficeFeed

```js
OfficeFeed.useSimulation(false);   // 关闭随机模拟

OfficeFeed.addToken('pump', {      // 'pump' | 'fomo' | 'gmgn'
  sym: 'ABC', mcap: 32000, holders: 410,
  smart: 3, bond: 64, safe: 72,    // bond 只对 pump 有意义，0-100
  born: Date.now() - 20000
});

OfficeFeed.addWallet({
  src: 'gmgn', sym: 'ABC', kind: '聪明钱', // 聪明钱|狙击手|巨鲸|KOL|新钱包
  addr: '7xKp…9fQa', act: 'buy',           // 'buy' | 'sell'
  usd: 5200
});

OfficeFeed.setPrice('BTC', 98500);         // 二级币价，sym 要在 COINS 列表里
```

要加新币种，改 `index.html` 里的 `COINS` 数组。

## 4. 数据源（接入前请以官方文档为准）

| 模块 | 建议来源 | 说明 |
|---|---|---|
| PUMP 新币、交易、毕业 | PumpPortal WebSocket（`wss://pumpportal.fun/api/data`） | 常用订阅：新币、代币交易、账户交易、迁移（毕业）。具体方法名以其文档为准 |
| GMGN 聪明钱、热门榜 | GMGN 官方 OpenAPI | 需要申请 API key，接口以 GMGN 文档为准 |
| FOMO 热点 | 我没有确认到公开 API | 先向对方确认；没有的话，用 DexScreener 热门/推广榜加自己的钱包监控代替，卡片名称可保留 |
| 钱包追踪（Solana） | Helius 或其他 RPC 的 webhook / logsSubscribe | 维护一张“聪明钱/巨鲸/KOL 地址表”，命中就推 `addWallet` |
| 二级主流币 | Binance WebSocket 行情流或 CoinGecko `/coins/markets` | 主流币实时用交易所 WebSocket，热门币可用 CoinGecko |
| 二级热门币、DEX 价格 | DexScreener API | 免费，有频率限制 |

## 5. 安全分数（VET）怎么算

小样里的 `safe` 是随机数。真实版建议由后端按规则算，例如：

- 流动性是否达到下限
- 前 10 持仓占比是否过高
- 持有人数量与增长
- 是否有明显貔貅或增发特征
- 创建者历史

规则写成配置文件（类似原帖的 `thresholds.py`），前端只显示结果。

## 6. 视觉扩展

已完成（v2）：
1. 蜘蛛抓取失败变红，标出红叉并返回巢穴，CRAWLER 卡片显示失败次数
2. 钱包动态触发时，从 WHALE RADAR 拉光束到对应来源的工位，并显示买卖金额
3. PUMP 新币毕业时，一枚 P 币从 PUMP 工位飞向 DEX 口，标注“毕业 → 二级”
4. 点击办公室里的智能体，或顶部智能体卡片，弹出它最近 20 条决策日志

后续可做：
1. 真实数据下，失败由接口超时、限流触发，而不是随机
2. 毕业的币自动加入二级热门币卡片
3. 加音效开关（需用户点击后才能播放）

## 7. 给 Muse 的提示词（可直接粘贴）

> 我有一个单文件前端 `index.html`（链上雷达办公室，Canvas 动画 + 三个来源卡片 PUMP/FOMO/GMGN + 钱包追踪 + 二级币价）。请在不改变视觉风格的前提下：
> 1. 新建一个 Node.js 后端，用 WebSocket 向前端推送 JSON 事件（token / wallet / price 三类）。
> 2. 接入 PumpPortal WebSocket 获取 pump 新币和交易；接入 CoinGecko 或 Binance 获取二级币价。
> 3. 前端收到事件后调用 `window.OfficeFeed.addToken / addWallet / setPrice`，并先调用 `OfficeFeed.useSimulation(false)`。
> 4. API key 放 `.env`，不要进前端代码。
> 5. 先做断线重连和限流，再做 GMGN、钱包地址表。

## 8. 风险提示

- 这是可视化与监控工具，不构成投资建议。
- 新币（尤其是发射台新币）波动和归零风险极高；“聪明钱”标签和安全分数都可能出错。
- 不要把私钥放进前端或提交到代码仓库。若以后要接自动下单，务必先做模拟盘、单笔上限和总亏损熔断。

## 9. v3 升级记录（2026-10-21，多米按本教程实施）

已完成本教程的目标：小样升级为接真实数据的版本。改动点：

- **音效**：Web Audio 纯合成，无外部音频文件。蜘蛛抓包"嗒"、送达 HUB"叮"、钱包买入上扬/卖出下坠、抓取失败低鸣、毕业四音琶音、智能体说话轻音（2.5 秒节流）。右上角音效开关，默认开（浏览器自动播放策略要求用户先点一次页面）。
- **PUMP**：直连 `wss://pumpportal.fun/api/data`，`subscribeNewToken` + `subscribeMigration` 免费订阅。bond% = vSolInBondingCurve/85 真算；新币/分钟为真实 60 秒窗口计数；断线指数退避重连。
- **FOMO**：无公开 API，诚实标注。用 DexScreener `/token-boosts/top/v1`（付费热度榜）+ `/token-profiles/latest/v1`，60 秒轮询。
- **GMGN**：官方 API 未开放。从 gmgnai-wrapper 仓库挖到其前端聪明钱钱包榜接口（`.../rank/sol/wallets/1d`），经 allorigins 代理抓取；代理不可用时（实测 corsproxy/codetabs/allorigins 均不稳定）自动降级为 DexScreener Solana 新币榜，页面明确标注降级状态。
- **主流币+热门币**：CoinGecko `/coins/markets`（10 币，含 7 天 sparkline），60 秒轮询。币安 API 在云主机被地区锁，未采用。
- **榜单列调整**：假"安全分"/"聪明钱数"列移除，换成真实的"流动性"（DexScreener）列；GMGN 切为聪明钱钱包榜（地址/标签/1d 收益/买/卖/胜率）。
- **验证**：全部接口用 curl/Node/Python 实测；三种渲染路径（PUMP 行、GMGN 钱包行、GMGN 降级行）用 mock 数据截图验收；另修了 `spark()` 空数组导致的 SVG path 非法报错、WIF 的 CoinGecko ID（`dogwifcoin`）。
- `window.OfficeFeed` 四个接口保持不变，内部走真实通道。

## 10. v3.1 升级记录（2026-10-21，用户追加需求）

新增两个智能体：BTCW（BTC 鲸眼）、ETHW（ETH 鲸眼）。

- **数据源**：Coinbase Exchange 公开成交流。主通道 `wss://ws-feed.exchange.coinbase.com` 的 `matches` 频道（BTC-USD/ETH-USD，无需 key，浏览器可直连）；WS 连续失败 3 次后自动降级为 REST 轮询（`/products/{BTC,ETH}-USD/trades`，15 秒一次，水位线去重，首轮只建水位不触发）。
- **阈值**：大单 BTC ≥$100K / ETH ≥$50K；巨鲸 BTC ≥$500K / ETH ≥$250K。
- **音效**：大买单放烟花（升空哨 + 12 颗随机高音炸开），大卖单爆炸声（130→26Hz 低频下坠 + 低通噪声轰）。只有巨鲸档才弹智能体气泡（"BTC 巨鲸大卖 $675.2K"），大单档只闪卡 + 音效，避免刷屏。
- **画面**：鲸鱼雷达（右上）处爆发——买单是 46 粒五彩烟花粒子（带重力），卖单是红色冲击波圆环；新智能体在画布上有自己的工位（BTCW 橙色、ETHW 蓝色）。
- **卡片**：显示实时价 + 24h 涨跌（走 CoinGecko 轮询）+ 大买/大卖计数；能量条 = 大买占比。
- 桌面端实测注意：云主机出口代理会掐 WebSocket（`InvalidUpgrade: invalid Connection header: close`），属测试环境假象，用户手机直连不受影响；验收用 mock 数据走 DOM + 截图。
