# 多米链上雷达办公室

单文件、零构建、打开即用的链上实时监控页。10 个智能体在"办公室"里跑来跑去，实时盯盘，还有音效。

![多米链上雷达办公室](screenshot.png)

## 怎么玩

1. 下载 `index.html`，双击用浏览器打开（手机 / 电脑都行）
2. 点一下页面任意位置激活音效（浏览器自动播放策略要求）
3. 右上角可随时开关音效

## 监控内容

| 板块 | 数据 | 来源 |
|---|---|---|
| PUMP | pump.fun 新币实时推送 + 毕业事件，bond 进度真实计算 | PumpPortal WebSocket |
| FOMO | 付费热度榜 / 最新上币，60 秒轮询 | DexScreener |
| GMGN | 聪明钱钱包榜；代理不可用时自动降级为 DexScreener Solana 新币并明确标注 | GMGN 前端接口（经代理） |
| 二级 | BTC / ETH / SOL / BNB / XRP + DOGE / PEPE / WIF / BONK / TRUMP，60 秒轮询 | CoinGecko |
| BTC 鲸眼 | Coinbase 实时成交流，大买单放烟花 | Coinbase |
| ETH 鲸眼 | Coinbase 实时成交流，大卖单爆炸声 | Coinbase |

**鲸鱼阈值**：大单 BTC ≥ $100K / ETH ≥ $50K；巨鲸 BTC ≥ $500K / ETH ≥ $250K（巨鲸档弹气泡播报）。

**音效**（Web Audio 实时合成，零音频文件）：蜘蛛抓包、数据送达、钱包买卖、抓取失败、毕业琶音、大买单烟花、大卖单爆炸。

## 本地开发

```bash
# 直接打开就行，不需要构建
open index.html
```

`window.OfficeFeed` 暴露了数据接口（`addToken` / `addWallet` / `setPrice` / `useSimulation`），方便二次开发对接自己的数据源。详细升级思路见 [TUTORIAL.md](TUTORIAL.md)。

## 免责声明

仅供学习演示，不构成投资建议。

## License

MIT — 详见 [LICENSE](LICENSE)。
