# RackNerd 部署 OpenClaw 要买多大内存？一键装 8 GB、手动装 2 GB 起：档位对照、安装步骤与安全设置

RackNerd 能装 OpenClaw，买多大取决于你走哪条安装路径。ctrl 面板里的一键镜像按 **4 vCPU / 8 GB 内存** 起步，RackNerd 官方 Specials 页也直接写着 OpenClaw 至少要 8 GB；换成官方 `install.sh` 手动装，2 核 2.5 GB 的机器已经有人跑通。OpenClaw 本体很轻，真正吃资源的是你打算在本地跑模型，还是只调 API。

这两条路的差价不小，选错了要么白花钱，要么买完发现一键装根本点不下去。下面把档位、安装步骤、以及前排教程基本不提的安全设置一次讲清。

## 选购速览

> 想用 ctrl 一键装，直接上 **8 GB 档**，省下来的时间比省下的钱值；愿意 SSH 手动装、模型走 API，**2 GB 档就够跑**，4 GB 更宽裕。RackNerd 官方知识库明确写了**不提供退款、没有退款保证**，所以档位要在下单前定好。

RackNerd 的特价档位按批次放出，年付折算下来通常比月付便宜不少，具体型号和价格以下单页显示的为准。

👉 [查看 RackNerd 当前特价档位与价格](https://bit.ly/RacKnerd)

## 各内存档位能跑什么：按官方 ctrl 门槛对照

这张表按 RackNerd 官方 ctrl 一键应用目录标注的最低配置整理，用来判断"我这一档够不够"，比按价格猜靠谱。

| 内存档位 | ctrl 一键装跑得动的应用 | 手动装 OpenClaw | 适合谁 | 入口 |
| --- | --- | --- | --- | --- |
| **1 GB** | WireGuard (wg-easy)、Webmin、CyberPanel（均 1 vCPU / 1 GB） | 不建议，Node.js 22 装依赖时容易吃紧 | 只做内网穿透、面板管理 | [看这一档](https://bit.ly/RacKnerd) |
| **2 GB** | n8n、Coolify、Portainer、AdminBolt、CloudPanel（2 vCPU / 2 GB） | 可行，有教程在 2 核 2.5 GB 上装通 | 手动装 OpenClaw、跑工作流 | [看这一档](https://bit.ly/RacKnerd) |
| **4 GB** | Hermes Agent、AnythingLLM、Ollama、LibreChat（2 vCPU / 4 GB） | 宽裕，留得下 Docker 和日志 | 想同时跑本地小模型或其他 AI 应用 | [看这一档](https://bit.ly/RacKnerd) |
| **8 GB** | OpenClaw 一键镜像（官方标 4 vCPU / 8 GB） | 明显过量，除非还要跑 Ollama | 不想碰命令行、要官方镜像开箱可用 | [看这一档](https://bit.ly/RacKnerd) |

表格里没有价格，因为 RackNerd 的 Specials 是按批次更新的，写死的数字过两天就会误导人。点进去看到的才是当前有效的。

## 一键装还是手动装，这条官方警告先看

ctrl 一键装 OpenClaw 的流程是：在 Specials 页下单 → 登录 ctrl 面板 → 打开 One-Click Apps → 选 OpenClaw → 面板重装系统并部署，大约 **15 到 45 分钟** → 装完在 ctrl 里显示 SSH root 和 app 登录凭据。

官方对这一步的说明写得很直白：一键安装会**永久抹掉 VPS 上现有的数据**（系统、应用、文件），然后部署一个带所选应用的全新镜像。机器上已经有东西的，先备份再点。

手动装走的完全是另一条路。官方脚本 `curl -fsSL https://openclaw.ai/install.sh | bash` 会自动装好 **Node.js 22+** 和依赖，两三分钟跑完，系统选 Debian 13 或 Ubuntu 22.04 / 24.04 都行。一键装是可选功能，RackNerd 的机器本身是不带管理的 KVM VPS，给完整 root SSH，你想怎么装就怎么装。

愿意自己动手的话，没必要为镜像多买 6 GB 内存：

👉 [按 2 GB 或 4 GB 档开通，自己跑安装脚本](https://bit.ly/RacKnerd)

## 手动部署 OpenClaw：从脚本到 Telegram 配对

有教程作者在 RackNerd 2 核 2.5 GB、LADC02 机房、Debian 13 的机器上完整走通过一遍，流程可以压成下面几步。

1. **跑安装脚本**。`curl -fsSL https://openclaw.ai/install.sh | bash`，方向键选 Yes，模式选 **QuickStart**。
2. **选模型**。Qwen 那一项官方脚本的原话是 "Qwen is free to use. First, get Openclaw up and running."，回车后要在浏览器打开 OAuth 链接完成登录，用 Google 账号最省事。想用国内可付费的方案，可以先去 platform.minimaxi.com 拿 Minimax 的 API Key；用 Anthropic 的话，配置里写 `anthropic/claude-sonnet-4-6` 这类短别名即可，OpenClaw 会把带日期的模型 ID 归一化。
3. **建 Telegram bot**。渠道选 Telegram，去 TG 找 **@BotFather**。认准名字后有蓝色对勾、账号 ID 为 **93372553** 的那个，先点 START 再发 `/newbot`。显示名可以用中文或 emoji，用户名必须是英文字母并以 `bot` 结尾。它返回的那串 token 就是 bot 的密码，直接粘进终端。
4. **skill 和 hooks**。第一次装 skill 选 No，之后可以让 OpenClaw 自己装；hooks 三个选项全勾上（方向键移动、空格选中、回车确认）。
5. **配对**。回 TG 给刚建的 bot 点 START，它会回一个配对码，在 VPS 终端执行 `openclaw pairing approve telegram <配对码>`。别把"配对码"三个汉字照抄进去。

装完 OpenClaw 的网关默认监听 **18789** 端口，配置文件在 `~/.openclaw/openclaw.json`，工作区是 `/root/.openclaw/workspace`。常用命令有 `openclaw status`、`openclaw logs`、`openclaw restart`、`openclaw update`。

微信这块要提前说清：**个人微信不支持**，微信个人版没有开放 Bot API。只有企业微信能通过第三方插件 `openclaw-plugin-wecom` 接入，需要 CorpID、AgentID、Secret，还要企业认证，新手直接放弃这条路，用 Telegram 体验最好。

## 装完必须做的安全设置

这节是大多数教程跳过、但最该读的部分。安全研究者发现过**数以万计**的 OpenClaw 安装可以从公网直接访问，其中一部分泄露了凭据和聊天记录；早期版本信任本地连接、不设密码，代理配置错一下，外部流量就会被当成内网流量。安全公司还在 OpenClaw 公共技能注册表里报告过**数百个**带恶意代码的 skill，有的把密码窃取器伪装成安装步骤。提示注入到今天仍**没有完整解法**：网页、邮件、文档里的一句话就能指挥 agent 泄露密钥或执行命令，配合长期记忆，植入的指令可以潜伏到以后再触发。

可执行的规则不多，但这几条别省：

- **给它单独一台机器**。别在跑 OpenClaw 的 VPS 上放别的重要业务，也别在上面存其他服务器的登录态。
- **别用 root 跑 agent**。用普通用户，并且不要把该用户加进 docker 组，那等价于 root。
- **控制面板只绑 `127.0.0.1`**，通过 SSH 隧道或 Tailscale、ZeroTier、Cloudflare Tunnel 这类内网穿透访问。**绝对不要把 18789 端口直接开到公网**。
- **即使只在内网也开认证**。网关 token 或密码该设就设。
- **只允许你自己的聊天账号**，把 user ID 限制住，否则谁找到这个 bot 都能指挥它。
- **用单独、限额、可轮换的 API Key**，专为 agent 建一个，出问题直接吊销。
- **工具从少到多**。先只给搜索和工作目录，shell、邮件、日历需要时再加，能只读就只读。
- **每个 skill 都看一眼代码和发布者**。装 skill 和装陌生人的程序是一回事。

真出问题时，日志里最常见的两句话是 `missing scope: operator.read` 和 `SecretRefResolutionError`，原因见下一节。

## 真实成本：服务器只是账单里最小的一块

跑 OpenClaw 的钱分三份：VPS 本身、模型调用费、你的时间。

VPS 那份最好算，年付特价档折算到每月通常是个位数美元。模型调用那份最容易失控：agent 会循环，一个看起来简单的任务可能触发十几次模型调用，跑偏的任务花掉的钱可以远超预期。开始之前就去模型提供商后台设一个月度硬上限，这是唯一有效的止损手段。

第三份是时间。安装半小时，之后是每周看更新、读日志、检查它有没有动过不该动的文件。这部分不体现在账单上，但决定了你适不适合养一个 24 小时在线的 agent。

顺便说一句，RackNerd 的 ctrl 目录里还有 Hermes Agent，最低 **2 vCPU / 4 GB**，由 Nous Research 出品，MIT 许可，更轻、不需要开放端口。想要一个更温和的起点，先在 4 GB 档上跑 Hermes Agent，玩熟了再上 OpenClaw，也是合理路径。

👉 [用年付特价开一台独立机器专门跑 agent](https://bit.ly/RacKnerd)

## 几个会让人卡住的运维细节

**关掉 SSH 服务就死**。OpenClaw 装的是用户级 systemd 服务，不执行 `loginctl enable-linger root`，最后一个 SSH 会话关掉，网关就停了。现象是"刚才还能用，现在连不上"。

**token 传不进服务**。配置里用 `${OPENCLAW_GATEWAY_TOKEN}` 引用环境变量时，systemd 单元必须用 `EnvironmentFile` 显式传进去，否则服务启动几次后失败，`journalctl --user -u openclaw-gateway` 里是 `SecretRefResolutionError`。日志用 `journalctl --user -u openclaw-gateway -f` 看。

**`missing scope: operator.read`**。裸 OpenClaw 网关在 WebSocket 握手时要求 **Ed25519 设备身份**，缺了会"连接成功但授予零权限"。别去开 `dangerouslyDisableDeviceAuth`，官方文档称其为严重的安全降级。

**用 Caddy 反代后设备变成"远程设备"**。Caddy 会加 `X-Forwarded-For`，OpenClaw 把它当真实客户端，后台设备就需要批准一次：`openclaw devices approve <requestId> --token ...`。

**升级会丢配对设备**。部分版本的更新会重写配置、清掉已配对设备列表，大版本升级前先把 `~/.openclaw/` 整个备份。

**证书申请失败别反复试**。Let's Encrypt 连续失败 5 次会限流 1 小时。Caddy 第一次启动前三件事必须同时对：DNS 的 A 记录指向真实 IP、80 端口对 `0.0.0.0/0` 开放、443 端口开放。

## FAQ

**1 GB 内存能跑 OpenClaw 吗？**
手动装勉强，Node.js 22 拉依赖和构建时容易内存不足。官方一键镜像要 8 GB，手动装的经验值从 2 GB 起。

**必须用 Telegram 吗？**
不是。OpenClaw 还支持 WhatsApp、Discord、Slack 等渠道，配对逻辑类似。个人微信不行，企业微信要靠第三方插件。

**需要买域名、备案吗？**
只自己用的话，网关绑 `127.0.0.1`，走 SSH 隧道访问，不需要域名。RackNerd 的机房在境外，不涉及国内备案流程。

**续费会不会涨？**
特价档的计费周期和续费金额在下单页和账单里显示，付款前自己确认一遍是年付还是月付。RackNerd 官方知识库明确不提供退款，也没有退款保证，这一点比价格更值得先看清。

**装完怎么更新？**
`openclaw update`。OpenClaw 发安全修复的频率不低，每周看一次更新是基本操作。

**支付方式有哪些？**
官方知识库的支付方式条目以主要信用卡开头。如果你打算用支付宝或微信，在结账页自己确认一遍再下单。

## 最后怎么选

按情况对号入座：

- 不想碰命令行、要官方镜像开箱可用 → **8 GB 档 + ctrl 一键装**
- 会 SSH、模型走 API 不跑本地 → **2 GB 档手动装**，预算宽松就上 4 GB
- 想同时跑 Ollama、AnythingLLM 这类本地模型 → **4 GB 起**，模型越大越吃内存
- 只想先试试 agent 是什么感觉 → **4 GB 档装 Hermes Agent**，比 OpenClaw 温和

下单前确认两件事：机器上有没有不能丢的数据（一键装会抹盘），以及你打算跑的模型每月大概花多少钱（先去模型商后台设上限）。

👉 [打开 RackNerd Specials 选好档位，开始部署 OpenClaw](https://bit.ly/RacKnerd)
