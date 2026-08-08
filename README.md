# LadderTools

LadderTools 是一组面向 Clash/OpenClash、Stash 和 Shadowrocket 的分流配置。仓库不保存机场节点或私人订阅地址，只维护策略组、规则顺序和少量自定义规则集。

> 使用前请确认你拥有相关服务的合法使用权限，并遵守所在地区的法律法规。不要把带令牌的私人订阅 URL 提交到仓库、公开转换站日志或截图中。

## 目录与配置分类

```text
LadderTools/
├─ configs/
│  ├─ clash.ini                  # 推荐：通用 subconverter 转换模板
│  ├─ clash-multi-sub.ini        # 三订阅来源隔离的 subconverter 模板
│  ├─ stash.yaml                 # Stash 私人订阅包装示例
│  ├─ stash-policy.stoverride    # Stash 策略组及规则覆写
│  └─ shadowrocket.conf          # Shadowrocket 完整公共配置
├─ rules/                        # 本仓库自定义 classical 规则集
└─ docs/qrcode/                  # 推荐配置的 URL 二维码
```

### 配置文件适用范围

| 文件 | 类型 | 适用客户端/场景 | 是否含节点 | 推荐程度 |
| --- | --- | --- | --- | --- |
| `configs/clash.ini` | subconverter 模板 | 将私人订阅转换成 Clash/OpenClash 或 Stash YAML | 否，节点由转换请求的 `url` 传入 | 推荐 |
| `configs/clash-multi-sub.ini` | 多订阅 subconverter 模板 | 使用 `default/premium/budget` 三个 tag 合并订阅，并按来源建立独立策略组 | 否，订阅由转换请求的 `url` 传入 | 推荐 |
| `configs/stash.yaml` | Stash 基础 YAML 示例 | 把一个私人订阅包装成 `proxy-provider`，再叠加覆写 | 否，仅含占位订阅 URL | 推荐给不使用订阅转换的 Stash 用户 |
| `configs/stash-policy.stoverride` | Stash Override | 给已有 Stash 配置统一替换策略组、规则提供器和规则 | 否，复用原配置的 `proxies`/`proxy-providers` | 推荐 |
| `configs/shadowrocket.conf` | Shadowrocket 配置 | iOS/iPadOS Shadowrocket 远程配置或本地导入 | 否，节点订阅需另加 | 推荐 |
| `rules/*.list` | Clash classical/规则集文本 | 由以上配置远程引用，也可被兼容客户端单独引用 | 不适用 | 配套文件，不是完整配置 |

## 快速开始

### 方案 A：Clash/OpenClash 通过订阅转换使用

`clash.ini` 不是可以直接导入客户端的 YAML，而是 subconverter 的外部配置模板。将以下三个地址分别进行 URL 编码后交给可信的 subconverter：

- `url`：你的私人节点订阅地址；
- `config`：`https://raw.githubusercontent.com/mangataw/LadderTools/main/configs/clash.ini`；
- 转换服务地址：你自建或信任的 subconverter。

OpenClash 推荐参数：

```text
target=clash&expand=true&new_name=true
```

通用请求结构如下，`<>` 中的内容需要替换并 URL 编码：

```text
https://<subconverter-host>/sub?target=clash&url=<private-subscription>&config=<clash.ini-raw-url>&expand=true&new_name=true
```

将生成的订阅 URL 添加到 OpenClash，更新配置后检查策略组是否都有可用节点。`expand=true` 会把远程规则展开进最终配置，适合希望 OpenClash 获得一份自包含规则配置的场景。

若转换服务由第三方运营，运营方理论上能够看到 `url` 参数中的订阅令牌。私人订阅优先使用自建转换服务。

### 方案 B：Stash 使用订阅转换

仍使用 `clash.ini`，但推荐让转换结果保留 `rule-provider`：

```text
target=clash&expand=false&classic=true&new_name=true
```

生成后把转换 URL 作为 Stash 远程配置导入。`expand=false` 会让客户端按 `interval` 更新规则集；`classic=true` 用于兼容本仓库及第三方规则中 `DOMAIN-SUFFIX,...`、`IP-CIDR,...` 这类完整 Clash 规则行。

### 方案 C：Stash 不经过订阅转换

1. 复制 `configs/stash.yaml` 到私有位置，将 `proxy-providers.subscription.url` 替换为自己的订阅地址。
2. 确认订阅返回的是 Clash/Stash YAML，并含顶层 `proxies` 字段。
3. 在 Stash 中导入该基础配置。
4. 再把 `configs/stash-policy.stoverride` 作为覆写添加并关联到该配置。
5. 更新配置和覆写，确认“手动切换节点”能看到订阅节点，然后再启用代理。

覆写文件使用 `#!replace` 替换 `proxy-groups`、`rule-providers` 和 `rules`，但不会提供节点。若基础配置没有 `proxies` 或 `proxy-providers`，所有依赖 `include-all: true` 的分组都会为空。

### 方案 D：Shadowrocket

1. 在 Shadowrocket 中通过 URL 或下载文件导入 `configs/shadowrocket.conf`。
2. 回到首页，单独添加自己的节点订阅并更新节点。
3. 打开配置并检查“节点选择”“手动切换节点”及各地区组是否能看到节点。
4. 首次启用后，分别测试国内直连、普通代理、AI 服务和一个流媒体站点。

该文件只包含通用网络参数、策略组和规则，不包含任何私人节点。地区组依赖节点名称中的“香港/HK”“日本/JP”“美国/US”等关键字；机场命名不匹配时，需要修改 `policy-regex-filter`，或直接使用“手动切换节点”。

## 配置二维码

二维码编码的是 GitHub Raw URL，不包含私人订阅。客户端支持“扫描配置 URL”时可直接扫描；若只打开浏览器，请复制打开后的 URL 到客户端。二维码跟随 `main` 分支，仓库变更只有在推送后才会在线生效。

| Clash 转换模板 | Stash 策略覆写 | Shadowrocket 配置 |
| --- | --- | --- |
| [原始文件](https://raw.githubusercontent.com/mangataw/LadderTools/main/configs/clash.ini) | [原始文件](https://raw.githubusercontent.com/mangataw/LadderTools/main/configs/stash-policy.stoverride) | [原始文件](https://raw.githubusercontent.com/mangataw/LadderTools/main/configs/shadowrocket.conf) |
| ![Clash 模板二维码](docs/qrcode/clash-template.png) | ![Stash 覆写二维码](docs/qrcode/stash-override.png) | ![Shadowrocket 配置二维码](docs/qrcode/shadowrocket-config.png) |

`clash.ini` 的二维码只代表 `config` 模板地址，不能替代带 `url` 参数的订阅转换结果。`stash.yaml` 含私人订阅占位符，因此没有提供可直接使用的二维码。

## 分组分类

### 基础选择与地区组

- `节点选择`：主要入口，默认可在自动测速、地区组、手动组和直连之间切换。
- `手动切换节点`：收纳全部节点，适合正则未覆盖或临时指定节点。
- `手动切换分组`：在各地区测速组之间手动选择。
- `自动选择`：对全部有效节点执行 `url-test`。
- `香港节点`、`日本节点`、`美国节点`、`台湾节点`、`狮城节点`、`韩国节点`、`澳洲节点`、`德国节点`、`荷兰节点`、`英国节点`、`爱沙节点`：按节点名正则筛选并测速。
- `非日节点`：排除日本相关名称，用于明确要求避开日本出口的规则。

### 默认代理服务

- `AI`：OpenAI、Claude、Gemini、Copilot 等，默认优先日本、美国、新加坡等地区。
- `电报消息`：Telegram 域名和 IP 段。
- `谷歌服务`：Google 与 FCM。
- `游戏平台`：游戏平台及相关服务。

### 流媒体

- `油管视频`、`奈飞视频`、`巴哈姆特`、`国外媒体`：按地区或解锁节点选择。
- `奈飞节点`：只收纳名称含 `NF`、`Netflix`、`解锁`、`Media` 等关键字的节点。

### 直连优先服务

- `微软云盘`、`微软服务`、`苹果服务`：默认 `DIRECT`，需要时可改为代理。
- `网易音乐`、`哔哩哔哩`、`国内媒体`、`全球直连`：默认直连。

### 拦截与兜底

- `广告拦截`、`应用净化`：默认 `REJECT`，误杀时可切换为 `DIRECT`。
- `漏网之鱼`：最终未命中流量，默认直连，可手动改成 `节点选择`。

## 自定义规则集

本仓库的六个 `.list` 文件均使用完整 Clash 规则行，因此在 Clash/Stash 中应按 `classical`/`clash-classic` 处理。

| 文件 | 目标策略组 | 当前适用范围 |
| --- | --- | --- |
| `rules/proxy_direct.list` | `全球直连` | DeepL 等希望强制直连的域名 |
| `rules/proxy_ai.list` | `AI` | OpenAI、Copilot/Bing AI、Claude、Gemini、Manus |
| `rules/proxy_us.list` | `美国节点` | 美国出口专用规则；当前仅预留，尚无实际匹配项 |
| `rules/proxy_japan.list` | `日本节点` | 日本出口专用站点，例如 DMM |
| `rules/proxy_japan_not.list` | `非日节点` | 明确避开日本出口的站点，例如 JavDB |
| `rules/proxy.list` | `节点选择` | 通用代理兜底及部分区域商城 |

添加规则时保持一行一条，例如：

```text
DOMAIN-SUFFIX,example.com
DOMAIN,api.example.com
DOMAIN-KEYWORD,example
IP-CIDR,203.0.113.0/24,no-resolve
```

不要在这些 `classical` 文件中只写裸域名；裸域名集合应单独建文件并将 provider 的 `behavior`/类型改为 `domain`。纯 CIDR 集合则应使用 `ipcidr`，并在引用处按需增加 `no-resolve`。

## 分组和规则集的必要引用逻辑

流量处理链可以概括为：

```text
订阅节点
  → 地区/自动测速组
  → 服务策略组
  → 规则集把流量交给服务策略组
  → GEOIP,CN
  → FINAL / MATCH
```

维护配置时必须同时满足以下关系：

1. **规则的目标必须存在。** `ruleset=AI,...`、`RULE-SET,...,AI` 中的 `AI` 必须与策略组名称完全一致，包括 emoji、空格和大小写。改组名时要同步修改所有引用。
2. **策略组引用的子组必须有定义。** 例如 `AI` 引用 `日本节点`，`日本节点` 又依赖订阅节点；任何一层为空或拼写不一致，最终规则即使命中也没有可用出口。文件中的书写先后通常不重要，但必须避免策略组互相循环引用。
3. **Stash 的规则提供器必须成对引用。** `RULE-SET,custom-ai,AI` 要求 `rule-providers.custom-ai` 存在，同时 `proxy-groups` 中也要存在 `AI`。
4. **规则类型必须匹配文件内容。** 完整规则行用 `classical`，裸域名列表用 `domain`，纯 IP/CIDR 列表用 `ipcidr`。类型错误通常会导致规则下载成功但解析失败或完全不命中。
5. **规则自上而下、首次命中即停止。** 仓库先放自定义规则，再放具体服务和媒体规则，然后是国内直连、通用代理，最后才是 `GEOIP,CN` 与 `FINAL`/`MATCH`。把宽泛规则提前会遮蔽后续的精确规则。
6. **IP 规则避免不必要解析。** Telegram CIDR、中国 CIDR、`GEOIP,CN` 等 IP 类规则使用 `no-resolve`，避免为了匹配 IP 规则额外触发 DNS 查询。
7. **最终规则必须唯一且位于末尾。** Clash 转换模板使用 `[]FINAL`，Shadowrocket 使用 `FINAL,漏网之鱼`，Stash 使用 `MATCH,漏网之鱼`；其后不应再添加规则。

一个新增服务的最小改动通常包含三部分：

```text
① 新建或引用规则集
② 新建同名/明确映射的策略组，并保证它能到达节点或已有地区组
③ 在最终规则之前加入“规则集 → 策略组”的引用
```

若只做其中一步，常见结果是规则无法解析、策略组为空，或流量继续落入“漏网之鱼”。

## 关键参数说明

### `clash.ini` / subconverter

| 参数 | 作用 |
| --- | --- |
| `enable_rule_generator=true` | 让 subconverter 根据 `ruleset=` 生成规则 |
| `overwrite_original_rules=true` | 用模板规则覆盖订阅原有规则，避免两套规则顺序混杂 |
| `ruleset=策略组,类型:URL,86400` | 指定规则来源、目标策略组，以及 provider 模式下的更新周期 |
| `custom_proxy_group=...` | 创建选择、测速或节点正则筛选组 |
| `expand=true` | 转换时展开规则内容，常用于 OpenClash |
| `expand=false` | 保留远程 rule-provider，常用于 Stash |
| `classic=true` | 以 classical 方式处理完整 Clash 规则行 |
| `new_name=true` | 启用转换端的新名称处理；具体命名效果取决于 subconverter 版本 |

`url-test` 默认使用 `https://www.gstatic.com/generate_204` 检测，主要地区组每 600 秒测试一次；通常容差为 10 ms，美国组为 20 ms。数值过小会频繁切换，过大则可能长期停留在较慢节点。

### `stash.yaml`

| 参数 | 当前值 | 作用 |
| --- | --- | --- |
| `mode` | `rule` | 按规则分流 |
| `log-level` | `info` | 日常日志级别 |
| `proxy-providers.subscription.url` | 占位 URL | 必须替换为私人订阅 |
| `interval` | `3600` | 每小时更新一次节点订阅 |
| `benchmark-url` | `generate_204` | 节点可用性检测地址 |
| `benchmark-timeout` | `5` | 检测超时 5 秒 |

### `stash-policy.stoverride`

- `#!replace`：完全替换对应段落；基础配置原有的同名段落不会保留。
- `include-all: true`：从基础配置的全部节点及 provider 中收集节点。
- `filter`：使用正则筛选地区或解锁节点。
- `interval: 600`：策略组测速周期。
- `rule-providers.*.interval: 86400`：规则集每 24 小时更新。
- `behavior` 与 `format: text`：声明远程规则的语义类型和文本格式。

### `shadowrocket.conf`

- `bypass-system=true`：减少部分系统服务经代理时的兼容问题。
- `skip-proxy`、`tun-excluded-routes`：旁路局域网、回环、多播及保留地址。
- `dns-server`：同时配置腾讯/阿里 DoH 和国内普通 DNS；`fallback-dns-server=system` 作为回退。
- `ipv6=true`、`prefer-ipv6=false`：启用 IPv6，但仍优先使用 IPv4 结果。
- `hijack-dns`：接管发往 Google DNS 53 端口的查询。
- `udp-policy-not-supported-behaviour=REJECT`：节点不支持 UDP 时拒绝流量，避免意外直连泄漏。
- `block-quic=all-proxy`：阻断走代理的 QUIC，促使相关连接回退到可代理的 TCP/TLS。
- `interval=600`、`tolerance=10/20`、`timeout=5`：测速频率、切换容差和超时。

DNS、IPv6、QUIC 和 UDP 参数会直接影响局域网发现、游戏、视频和推送。若要调整，建议一次只改一个参数并保留可回滚副本。

## 维护建议

- 自定义精确规则放在通用第三方规则之前；`FINAL`/`MATCH` 始终保留在最后。
- 增删策略组后，同时搜索 `ruleset=`、`RULE-SET`、`proxy-groups` 和各组的子组引用。
- 外部规则依赖 GitHub Raw 可用性及上游目录结构；上游改名或删除会导致更新失败。
- 节点名称正则无法覆盖所有机场命名，新增地区关键字时同步修改 Clash、Stash 和 Shadowrocket 三套配置。
- 提交前至少检查：规则 URL 可访问、provider 类型正确、所有目标组存在、所有子组非空、最终规则唯一。
