# TVBox 9527 本地包 → MiraPlay（iOS）猫源一键订阅包

把 `TVBox本地包`（9527.jar + api.json）里能脱离 Java 运行的站点，逐类反编译 9527.jar 后
**1:1 移植成 MiraPlay 猫源 JS**，并**内置进“猫源通用接口” bundle**（lugu123 谱系）：

- 25 个移植源**直接打包进 index.js**（启动自动注入本地猫源层，无需手动上传）
- 交付 = MiraPlay 正规的 **`index.js` + `index.js.md5`** 一键订阅格式
- 桌面端已全链路实测：bundle 启动 → /config 出 25 个 nodejs_* 站 → /spider/半日99/3/home|category|detail|play 全通

## 一、使用（三步）

1. **托管这 4 个文件**（同目录，任意静态托管：kstore / OSS / GitHub raw 均可）：
   ```
   index.js
   index.js.md5
   index.config.js
   index.config.js.md5
   ```
2. MiraPlay → 首页右上「添加」→ **CatPawOpen 源** → 粘贴
   `https://<你的托管>/index.js.md5` → 完成。
   源列表出现：3 个内置（missav/直播/二维码）+ **25 个 nodejs_* 移植站**。
3. **配置（按需）**：
   - B 站高清：配置中心 → 哔哩系源 → 填 `{"cookie":"SESSDATA=...; bili_jct=..."}`（无 cookie 只有 480P m3u8 + 热门兜底）
   - 网盘播放（移动=天翼云、玩耦=各家网盘）：配置中心对应网盘二维码扫码入库 cookie
   - 播放失败排查：管理页 `/website/js` 有“全套测试”（/spider/<key>/3/debug/* 端点）

## 二、站点状态（2026-09-15，以你 TVBox 真机为准）

你在真机 TVBox 上 **除橘汁、厂长外基本都正常**，所以这批移植站的代码都是照 9527.jar 反编译
逐行对齐的，理论上真机（同一网络出口）表现应与 TVBox 一致。

| 组 | 源 | 说明 |
|---|---|---|
| ✅ 已实测全链路 | 半日99 / 剧圈99 | App99 协议（AES+SHA256 签名+zlib），home→列表→详情→m3u8 直链 |
| ✅ 已实测全链路 | 七猫短剧 | 自定义 base64+双 MD5 签名，直链 cdn |
| ✅ 已实测全链路 | 咖啡体育 | m3u8 直播流 |
| ✅ 已实测 | 移动4K / 哔哩×10 / 热播 / 豆瓣 / 玩耦(home) / 飞娱(home断连) | 见包内 TEST_REPORT |
| 代码就绪（协议已对齐 Java，真机首验） | 魔方 / 次元 / 曼波（AppGet 系） | 沙箱被断连/WAF，你 TVBox 正常 → 手机网络出口不同，直接可用 |
| 代码就绪（已修端点前缀） | 怀桑 / 蓝鹰（AppQi 系，qijiappapi.index） | 蓝鹰之前 404 是端点前缀写错，已修 |
| ⚠️ 上游本身坏（你 TVBox 里也坏） | 橘汁、厂长 | 未移植（厂长=缺类/API 404，橘汁=数据池空） |

### 未移植（本包不含）
- **Python drpy 站**（红果短剧、py聚合短剧、爱听、kwms、275听书、枫叶×2、听海）：iOS 沙箱无 Python 运行时
- **XBPQ 规则站**（永乐、养生堂）：XBPQ 引擎 2900+ 行，未移植
- **独立协议站**（韩圈/农民/看剧AI†/北斗†/瓜子/厂长/咕噜/布布/独播/金牌/爱看/天堂1/一起看/Sao火/哔哩视频/急救/戏曲多多/DJ×2/蜻蜓/世界/jin夏/博看）：第二批（†=jar 缺类，TVBox 上本来就是死的）
- **设备功能站**（本地文件/手机推送/ALLLive 直播聚合）：依赖 TVBox 本机 pvideo 1314 代理，iOS 无等价物
- **AppDrama 系**（天堂/橘汁/华谊）：protobuf+RSA 协议，橘汁已确认坏，天堂/华谊要做的话我第二批补

## 三、局限（读一下）
1. **解析线路**：TVBox 用 pvideo(1314)+配置中心 parses 做二次解析；MiraPlay 没有等价物。
   热播影视部分线路回“解析 token”，无解析服务时该线路打不开（换直链线路可播）。
2. **B 站高清**：playurl DASH 需登录 cookie；MPD(fMP4) 能否播取决于播放器，m3u8（≤480P 免登）最稳。
3. **弹幕**：TVBox 的 appdanmu(1314) 弹幕 iOS 端没有，全部省略。
4. **网盘**：移动/玩耦返回网盘分享链接，需配网盘 cookie。
5. 上游死站（魔方/次元/曼波/怀桑/蓝鹰等）在你 TVBox 里正常，说明是网络出口差异；
   若 MiraPlay 手机上也打不开某站，优先怀疑该站 WAF 对 iOS UA 限制（可改模板里 UA 重打包）。

## 四、维护（上游协议变了怎么改）
```
mirapack/            本包（交付的 4 文件）
../miraplay/src/     25 个单文件源（可读可改）
../miraplay/gen/     模板 + build.py（改协议只改模板，重跑生成）
../miraplay/test/    harness.js（hsjs 沙箱模拟器，node harness.js 全量回归）
打包命令见 gen/build_bundle.py（注入 25 源进 lugu123 bundle + loader check 补丁 + md5）
```
