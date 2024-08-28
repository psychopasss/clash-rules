# 简介

本项目生成适用于 [**Clash Premium、 Mihomo 内核**](https://github.com/MetaCubeX/mihomo/releases)的规则集（RULE-SET），同时适用于所有使用 Clash Premium、Mihomo 内核的 Clash 图形用户界面（GUI）客户端。使用 GitHub Actions 北京时间每天早上 6:30 自动构建，保证规则最新。

## 说明

本项目规则集（RULE-SET）的数据主要来源于项目 [@Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat) 和 [@v2fly/domain-list-community](https://github.com/v2fly/domain-list-community)；[`Apple`](https://github.com/Loyalsoldier/clash-rules/blob/release/apple.txt) 和 [`Google`](https://github.com/Loyalsoldier/clash-rules/blob/release/google.txt) 列表里的域名来源于项目 [@felixonmars/dnsmasq-china-list](https://github.com/felixonmars/dnsmasq-china-list)；中国大陆 IPv4 地址数据使用 [@17mon/china_ip_list](https://github.com/17mon/china_ip_list)。

本项目的规则集（RULE-SET）只适用于 Clash **Premium**、Mihomo 版本。Clash Premium、Mihomo 相对于普通版，增加了 **TUN 增强模式**，能接管设备所有 TCP 和 UDP 流量。

## 规则文件地址及使用方式

### 在线地址（URL）

> 如果无法访问域名 `raw.githubusercontent.com`，可以使用第二个地址（`cdn.jsdelivr.net`），但是内容更新会有 12 小时的延迟。以下地址填写在 Clash 配置文件里的 `rule-providers` 里的 `url` 配置项中。

- [https://raw.githubusercontent.com/psychopasss/clash-rules/release/xxx.txt](https://raw.githubusercontent.com/psychopasss/clash-rules/release/xxx.txt)
- [https://cdn.jsdelivr.net/gh/psychopasss/clash-rules@release/xxx.txt](https://cdn.jsdelivr.net/gh/psychopasss/clash-rules@release/xxx.txt)

将上面链接的 **xxx** 替换为下面你需要的英文单词，比如 direct

目前支持：
  - direct（直连域名列表）
  - proxy（代理域名列表）
  - gfw（GFWList 域名列表）
  - reject（广告域名列表）
  - private（私有网络专用域名列表）
  - youtube（YouTube 域名列表）
  - openai（OpenAI 域名列表）
  - notion（Notion 域名列表）
  - icloud（iCloud 域名列表）
  - apple（Apple 在中国大陆可直连的域名列表）
  - tld-not-cn（非中国大陆使用的顶级域名列表）
  - telegramcidr（Telegram 使用的 IP 地址列表）
  - lancidr（局域网 IP 及保留 IP 地址列表）
  - cncidr（中国大陆 IP 地址列表）
  - applications.txt（需要直连的常见软件列表，**不推荐使用，classical效率太低**）

### 使用方式

要想使用本项目的规则集，只需要在 Clash 配置文件中添加如下 `rule-providers` 和 `rules`。

#### Rule Providers 配置方式

以下只列举了三种类型，注意 behavior 的差别，自己以此类推。

```yaml
rule-providers:
  gfw:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/psychopasss/clash-rules@release/gfw.txt"
    path: ./ruleset/gfw.yaml
    interval: 86400

  telegramcidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/psychopasss/clash-rules@release/telegramcidr.txt"
    path: ./ruleset/telegramcidr.yaml
    interval: 86400

  applications:
    type: http
    behavior: classical
    url: "https://cdn.jsdelivr.net/gh/psychopasss/clash-rules@release/applications.txt"
    path: ./ruleset/applications.yaml
    interval: 86400
```

#### 黑名单模式 Rules 配置方式（推荐）

- 黑名单模式，意为「**只有命中规则的网络流量，才使用代理**」，适用于服务器线路网络质量不稳定或不够快，或服务器流量紧缺的用户。通常也是**软路由用户、家庭网关用户**的常用模式。
- 以下配置中，除了 `DIRECT` 和 `REJECT` 是默认存在于 Clash 中的 policy（路由策略/流量处理策略），其余均为自定义 policy，对应配置文件中 `proxies` 或 `proxy-groups` 中的 `name`。如你直接使用下面的 `rules` 规则，则需要在 `proxies` 或 `proxy-groups` 中手动配置一个 `name` 为 `PROXY` 的 policy。

```yaml
rules:
  - DOMAIN,clash.razord.top,DIRECT
  - DOMAIN,yacd.haishan.me,DIRECT
  - RULE-SET,private,DIRECT
  - RULE-SET,reject,REJECT
  - RULE-SET,google,PROXY
  - RULE-SET,youtube,PROXY
  - RULE-SET,openai,PROXY
  - RULE-SET,gfw,PROXY
  - RULE-SET,telegramcidr,PROXY,no-resolve
  - MATCH,DIRECT
```

#### 白名单模式 Rules 配置方式

- 白名单模式，意为「**没有命中规则的网络流量，统统使用代理**」，适用于服务器线路网络质量稳定、快速，不缺服务器流量的用户。
- 以下配置中，除了 `DIRECT` 和 `REJECT` 是默认存在于 Clash 中的 policy（路由策略/流量处理策略），其余均为自定义 policy，对应配置文件中 `proxies` 或 `proxy-groups` 中的 `name`。如你直接使用下面的 `rules` 规则，则需要在 `proxies` 或 `proxy-groups` 中手动配置一个 `name` 为 `PROXY` 的 policy。
- 如你希望 Apple、iCloud 和 Google 列表中的域名使用代理，则把 policy 由 `DIRECT` 改为 `PROXY`，以此类推，举一反三。
- 如你不希望进行 DNS 解析，可在 `GEOIP` 规则的最后加上 `,no-resolve`，如 `GEOIP,CN,DIRECT,no-resolve`。

```yaml
rules:
  - DOMAIN,clash.razord.top,DIRECT
  - DOMAIN,yacd.haishan.me,DIRECT
  - RULE-SET,private,DIRECT
  - RULE-SET,reject,REJECT
  - RULE-SET,icloud,DIRECT
  - RULE-SET,apple,DIRECT
  - RULE-SET,google,PROXY
  - RULE-SET,youtube,PROXY
  - RULE-SET,openai,PROXY
  - RULE-SET,direct,DIRECT
  - RULE-SET,lancidr,DIRECT
  - RULE-SET,cncidr,DIRECT
  - RULE-SET,telegramcidr,PROXY,no-resolve
  - GEOIP,LAN,DIRECT
  - GEOIP,CN,DIRECT
  - MATCH,PROXY
```

## 致谢

- [@Loyalsoldier/geoip](https://github.com/Loyalsoldier/geoip)
- [@Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat)
- [@gfwlist/gfwlist](https://github.com/gfwlist/gfwlist)
- [@v2fly/domain-list-community](https://github.com/v2fly/domain-list-community)
- [@felixonmars/dnsmasq-china-list](https://github.com/felixonmars/dnsmasq-china-list)
- [@17mon/china_ip_list](https://github.com/17mon/china_ip_list)

## 项目 Star 数增长趋势

[![Stargazers over time](https://starchart.cc/psychopasss/clash-rules.svg)](https://starchart.cc/psychopasss/clash-rules)
