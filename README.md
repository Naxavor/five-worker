# CF Worker 五协议节点
* **VLESS\TROJAN\XHTTP\Shadowsocks\Socks5五协议**
## 1. 部署与功能说明

### 核心设置
* **XHTTP 协议**：若绑定自定义域名，必须在 Cloudflare 域名下的 **左侧网络菜单** 中开启 **gRPC** 功能，否则无法连通。
* **节点导入说明**：
    * **Shadowsocks**：无法直接导入，需查看生成的节点信息后手动配置。
    * **Socks5**：导入后信息可能不完整，需手动补充配置。
    * Shadowsocks配置图示![Shadowsocks配置图示](Shadowsocks.jpg)
    * socks5配置图示![socks5配置图示](sock5.jpg)
* **初始密码**：初次部署后访问 Worker/page 地址，会自动弹出设置密码的界面,用于生成动态UUID，TROJAN等密码，需设置KV，名称为KV。
* **访问密码**：初次部署后访问 Worker/page 地址，编辑配置页面可以设置访问密码，用于保护节点配置信息不会被别人随意修改，需设置KV。
### 路径功能
* `/Socks5`: 留空。
* `环境变量或路径/REMOTE_CONFIG`: 远程参数配置路径，设置后可通过远程文件动态更新节点参数。

---

## 2. 配置优先级与容错机制

本程序支持三种方式设置参数，加载优先级如下：

1.  **KV (键值存储)**
2.  **远程配置 (Remote Config)**
3.  **环境变量 (Environment Variables)**

**运行逻辑**：
* **KV 限制**：免费版 Cloudflare Workers 的 KV 有读取次数限制。当 KV 无法读取时，系统会自动退回到 **远程配置**。
* **兜底策略**：如果远程配置也失效，将使用 **环境变量** 保证节点基本运行。
* **免部署更新**：使用 KV 或远程配置修改参数时，**无需重新部署代码**。

---

## 3. 远程配置文件模板 (config.json)

您可以将以下 JSON 内容保存为 `config.json` 并上传至 GitHub、Gist 或其他直链地址，然后在 Cloudflare 环境变量中设置 `REMOTE_CONFIG` 指向该文件地址。

```json
{
  "UUID": "",
  "KEY": "",
  
  "TIME": "99999",
  "UPTIME": "0",

  "PROXYIP": "",
  "DNS64": "64:ff9b::/96",
  "SOCKS5": "",
  "GO2SOCKS5": "",
  "BAN": "",

  "URL302": "",
  "URL": "",

  "SUBNAME": "MyWorkerSub",
  "ADD": "[www.visa.com](https://www.visa.com).tw:443#台湾Visa, [2606:4700::]:443#IPv6官方",
  "ADDAPI": "[https://raw.githubusercontent.com/username/repo/main/ips.txt](https://raw.githubusercontent.com/username/repo/main/ips.txt)",
  "ADDNOTLS": "[www.visa.com](https://www.visa.com).sg:80#新加坡非TLS",
  "ADDNOTLSAPI": "",
  "ADDCSV": "[https://raw.githubusercontent.com/username/repo/main/speed.csv](https://raw.githubusercontent.com/username/repo/main/speed.csv)",
  "LINK": "vless://..., vmess://...",
  
  "CFPORTS": "443, 8443, 2053, 2083, 2087, 2096",
  "DLS": "8",
  "CSVREMARK": "1",
  "EX": "false"
}
================================================================================
                              参数详细说明表
================================================================================
| 参数名         | 必填 | 类型   | 说明                                        |
================================================================================
| UUID           | 是   | 字符串 | 主用户ID (只能是UUID格式)，也是 Trojan/SS 的密码 |
| KEY            | 否   | 字符串 | 动态UUID密钥，填写后会覆盖上方静态UUID            |
| TIME           | 否   | 数字   | 动态UUID的有效周期(天)，默认 99999                |
| UPTIME         | 否   | 数字   | 动态UUID在当天的更新时间(0-23点)                  |
--------------------------------------------------------------------------------
| PROXYIP        | 否   | 字符串 | 出站代理IP(解决CF脏IP)，支持IPv4/IPv6，逗号分隔   |
| DNS64          | 否   | 字符串 | NAT64地址(如 64:ff9b::/96)，用于IPv6访问IPv4资源  |
| SOCKS5         | 否   | 字符串 | 前置代理，格式: user:pass@host:port              |
| GO2SOCKS5      | 否   | 字符串 | 指定走SOCKS5的域名，支持通配符*，填"all in"全走   |
| BAN            | 否   | 字符串 | 禁止访问的域名黑名单，逗号分隔                    |
--------------------------------------------------------------------------------
| URL302         | 否   | 字符串 | 访问Worker主页时 302 跳转到的网址(优先级>URL)     |
| URL            | 否   | 字符串 | 访问Worker主页时 伪装反代的网址                   |
--------------------------------------------------------------------------------
| SUBNAME        | 否   | 字符串 | 订阅文件下载时的默认文件名                        |
| ADD            | 否   | 字符串 | 手动添加优选节点，格式: 地址:端口#备注            |
| ADDAPI         | 否   | 字符串 | 远程优选IP列表(TXT格式)的下载链接                 |
| ADDNOTLS       | 否   | 字符串 | 手动添加非TLS节点(80端口等)                       |
| ADDNOTLSAPI    | 否   | 字符串 | 远程非TLS节点列表的下载链接                       |
| ADDCSV         | 否   | 字符串 | CloudflareSpeedTest 测速结果 CSV 文件的链接       |
| LINK           | 否   | 字符串 | 硬编码节点链接(vless://等)，直接合并到订阅中      |
--------------------------------------------------------------------------------
| CFPORTS        | 否   | 字符串 | 订阅中包含的TLS端口，默认为443,8443等6个端口      |
| DLS            | 否   | 数字   | 速度阈值，CSV测速大于此值(MB/s)才显示节点          |
| CSVREMARK      | 否   | 数字   | CSV文件中备注所在的列号(索引从0开始)               |
| EX             | 否   | 字符串 | 是否开启 XHTTP 协议 (true/false)                  |
================================================================================

[注意]
1. JSON 文件不支持注释，请在实际使用 config.json 时删除所有非 JSON 内容。
2. 确保 JSON 格式正确（例如：最后一项后面不要加逗号）。
