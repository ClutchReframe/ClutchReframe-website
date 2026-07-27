# ClutchReframe 根域名网站部署指南

更新日期：2026-07-27

本网站使用 GitHub Pages 发布，并以以下地址作为唯一正式入口：

```text
https://clutchreframe.com/
```

`https://www.clutchreframe.com/` 只作为兼容入口，最终应自动跳转到根域名。

## 1. 固定部署信息

| 项目 | 值 |
| --- | --- |
| GitHub 组织 | `ClutchReframe` |
| 仓库 | `ClutchReframe-website` |
| 发布分支 | `main` |
| 发布目录 | `/ (root)` |
| Pages Custom domain | `clutchreframe.com` |
| Canonical | `https://clutchreframe.com/` |

仓库根目录中的 `CNAME` 必须只包含：

```text
clutchreframe.com
```

不要写入协议、路径、`www` 或其他域名。

## 2. 本地预览

在 Windows CMD 中运行：

```cmd
cd C:\Dev\ClutchReframe-website
py -m http.server 3000
```

然后访问：

```text
http://localhost:3000/
```

重点检查首页、`404.html`、`robots.txt`、`sitemap.xml` 和
`og-image.png`。结束预览时按 `Ctrl+C`。

## 3. 创建并推送 GitHub 仓库

在 `ClutchReframe` 组织下创建 Public 仓库：

```text
ClutchReframe-website
```

不要在 GitHub 创建仓库时自动添加 README、`.gitignore` 或 License，
以免与本地初始提交冲突。

在本地仓库中确认远程地址后提交并推送：

```cmd
cd C:\Dev\ClutchReframe-website
git remote add origin https://github.com/ClutchReframe/ClutchReframe-website.git
git add .
git commit -m "feat: launch ClutchReframe product hub"
git push -u origin main
```

如果 `origin` 已存在，先运行 `git remote -v`；只有地址不正确时才使用
`git remote set-url origin`。

## 4. 验证域名所有权

在更改 DNS 前，优先在 GitHub 的组织设置中验证 `clutchreframe.com`，
以降低自定义域名被其他 Pages 仓库占用的风险：

1. 登录 GitHub，打开
   <https://github.com/organizations/ClutchReframe/settings/pages>。
2. 在 **Verified domains** 区域点击 **Add a domain**。
3. 输入 `clutchreframe.com`，然后点击 **Add domain**。
4. GitHub 会显示一条 TXT 记录的 **Host** 和 **Value**。这个页面先不要关。
5. 新开一个浏览器标签页并登录
   <https://dash.cloudflare.com/>。
6. 点击域名 **clutchreframe.com**，再点击左侧 **DNS** → **Records**。
7. 点击 **Add record**，逐项填写：

   | 输入框 | 填写内容 |
   | --- | --- |
   | Type | `TXT` |
   | Name | 原样复制 GitHub 显示的 Host |
   | Content | 原样复制 GitHub 显示的 Value |
   | TTL | `Auto` |

8. 点击 **Save**。TXT 记录没有橙色云朵开关，这是正常的。
9. 回到 GitHub 的标签页，点击 **Verify**。
10. 如果暂时验证失败，等 5 至 30 分钟后再点一次 **Verify**，不要重复添加
    TXT 记录。

不要删除 GitHub 要求保留的 TXT 验证记录。

GitHub 官方的 Pages 域名验证说明：

<https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages>

## 5. 开启 GitHub Pages

打开：

```text
https://github.com/ClutchReframe/ClutchReframe-website/settings/pages
```

在 **Build and deployment** 中选择：

- Source：`Deploy from a branch`
- Branch：`main`
- Folder：`/ (root)`

保存后，在 **Custom domain** 中填写：

```text
clutchreframe.com
```

先将域名添加到 GitHub Pages，再配置 DNS。GitHub 的官方说明：

<https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site>

## 6. 配置 Cloudflare DNS

最省事的配置只需要 **5 条记录**：

- 根域名 `@` 的 4 条 A 记录。
- `www` 的 1 条 CNAME 记录。

第一次配置时可以跳过可选的 AAAA 记录。先让这 5 条记录和 HTTPS 正常工作，
以后再补 IPv6。

### 6.1 打开正确的 Cloudflare 页面

1. 登录 <https://dash.cloudflare.com/>。
2. 在账号首页点击 **clutchreframe.com**。不要点其他域名。
3. 先看 **Overview** 页面中的域名状态，必须是 **Active**。
4. 点击左侧 **DNS** → **Records**，进入 DNS 记录列表。

如果域名状态是 **Pending**，先按 Cloudflare 的提示到域名注册商更换
nameserver；在状态变成 **Active** 前，不要继续本节。

### 6.2 先检查有没有冲突的旧记录

在 DNS 记录列表上方的搜索框中搜索 `clutchreframe.com`，只检查 **Name
恰好等于 `clutchreframe.com`** 的记录：

- 保留 MX、TXT、CAA、NS 等记录。
- 如果已有指向其他网站的 A、AAAA 或 CNAME 记录，截图备份后删除这些旧的
  网站记录。
- 不要删除名称中带有 `clips`、`live`、`live-dota2` 或 `support` 的记录。

然后搜索 `www`：

- 如果没有 `www` 记录，直接进入下一步。
- 如果已有 `www` 的 A、AAAA 或 CNAME 记录，截图备份后删除它；稍后会添加
  唯一正确的 `www` CNAME。
- `www` 不能同时存在旧 A/AAAA 记录和新的 CNAME 记录。

Cloudflare 保存后可能把 Name `@` 显示成完整的
`clutchreframe.com`，这是正常的。

### 6.3 添加根域名的 4 条 A 记录

先添加第一条：

1. 点击 **Add record**。
2. **Type** 选择 `A`。
3. **Name** 输入 `@`。
4. **IPv4 address** 输入 `185.199.108.153`。
5. **Proxy status** 选择 **DNS only**。正确状态是灰色云朵，不是橙色云朵。
6. **TTL** 选择 `Auto`。
7. 点击 **Save**。

再点击三次 **Add record**，按完全相同的方式添加另外三个 IP。最终四条
A 记录必须是：

| Type | Name | IPv4 address | Proxy status | TTL |
| --- | --- | --- | --- | --- |
| A | `@` | `185.199.108.153` | DNS only（灰色云朵） | Auto |
| A | `@` | `185.199.109.153` | DNS only（灰色云朵） | Auto |
| A | `@` | `185.199.110.153` | DNS only（灰色云朵） | Auto |
| A | `@` | `185.199.111.153` | DNS only（灰色云朵） | Auto |

不要把 4 个 IP 填在同一条记录里；每个 IP 单独保存为一条 A 记录。

### 6.4 添加 www 兼容入口

1. 点击 **Add record**。
2. **Type** 选择 `CNAME`。
3. **Name** 输入 `www`。
4. **Target** 输入 `clutchreframe.github.io`。
5. **Proxy status** 选择 **DNS only**，确认是灰色云朵。
6. **TTL** 选择 `Auto`。
7. 点击 **Save**。

Target 中不要填写 `https://`，不要添加 `/`，也不要填写仓库名。

正确的 `www` 记录只有这一条：

| Type | Name | Target | Proxy status | TTL |
| --- | --- | --- | --- | --- |
| CNAME | `www` | `clutchreframe.github.io` | DNS only（灰色云朵） | Auto |

当 Pages Custom domain 使用 `clutchreframe.com` 且两组 DNS 都正确时，
GitHub Pages 会将 `www.clutchreframe.com` 自动重定向到根域名。

### 6.5 最后对照一次

在 Cloudflare 的 DNS 记录列表中搜索 `clutchreframe.com`，确认：

- 有 4 条根域名 A 记录，IP 与上表逐字一致。
- 有 1 条 `www` CNAME，Target 是 `clutchreframe.github.io`。
- 这 5 条记录的 Proxy status 全部是 **DNS only** 灰色云朵。
- 没有其他指向旧网站的根域名 A/AAAA/CNAME。
- 没有其他 `www` A/AAAA/CNAME。
- 第 4 节添加的 GitHub TXT 验证记录仍然存在。

只要有一条同名 A/AAAA 记录显示橙色云朵，就把同名记录全部改成
**DNS only**。不要开启 Cloudflare Proxy。

Cloudflare 官方的添加记录说明：

<https://developers.cloudflare.com/dns/manage-dns-records/how-to/create-dns-records/>

### 6.6 可选：添加 IPv6

这一步不是首次上线的必需步骤。前面 5 条记录正常后，如需 IPv6，再分别
添加以下四条 AAAA 记录：

| Type | Name | IPv6 address | Proxy status | TTL |
| --- | --- | --- | --- | --- |
| AAAA | `@` | `2606:50c0:8000::153` | DNS only（灰色云朵） | Auto |
| AAAA | `@` | `2606:50c0:8001::153` | DNS only（灰色云朵） | Auto |
| AAAA | `@` | `2606:50c0:8002::153` | DNS only（灰色云朵） | Auto |
| AAAA | `@` | `2606:50c0:8003::153` | DNS only（灰色云朵） | Auto |

每个 IPv6 地址也必须单独保存为一条记录。

### 6.7 不得修改的记录

不要修改或删除现有子域名记录，包括但不限于：

```text
clips.clutchreframe.com
live.clutchreframe.com
live-dota2.clutchreframe.com
support.clutchreframe.com
```

不要添加 `*.clutchreframe.com` 通配符记录，也不要改动邮件相关 MX、SPF、
DKIM 或 DMARC 记录。

### 6.8 看见报错时怎么处理

| 现象 | 处理方法 |
| --- | --- |
| 添加 `www` CNAME 时提示已有同名记录 | 搜索 `www`，删除旧的 `www` A、AAAA 或 CNAME，然后重新添加 |
| GitHub 显示 DNS check unsuccessful | 对照第 6.5 节检查 5 条记录，确认都是灰色云朵，然后等待 30 分钟再刷新 |
| Cloudflare 中 `@` 变成了 `clutchreframe.com` | 正常，不需要修改 |
| `www` 查询不到 CNAME | 确认 `www` 是 DNS only，而且没有同名旧记录；等待 DNS 缓存更新 |
| Enforce HTTPS 不能勾选 | 先不要改 DNS；证书签发最长可能需要 24 小时 |

## 7. HTTPS

1. 回到
   <https://github.com/ClutchReframe/ClutchReframe-website/settings/pages>。
2. 找到 **Custom domain**，确认内容仍是 `clutchreframe.com`。
3. 等待页面显示 **DNS check successful**。
4. 等待 **Enforce HTTPS** 复选框变成可点击状态。
5. 勾选 **Enforce HTTPS**。

DNS 和证书可能需要数分钟到 24 小时。等待期间不要反复删除 Custom domain、
切换橙色云朵或删除 DNS 记录。

## 8. 发布验证

在浏览器中检查：

```text
https://clutchreframe.com/
https://www.clutchreframe.com/
https://clutchreframe.com/404.html
https://clutchreframe.com/robots.txt
https://clutchreframe.com/sitemap.xml
https://clutchreframe.com/og-image.png
```

预期：

- 根域名首页返回 `200 OK`。
- `www` 自动跳转到 `https://clutchreframe.com/`。
- HTTPS 无证书警告。
- 首页的全部产品入口跳转到
  `https://live-dota2.clutchreframe.com/`。
- 现有子域名仍可独立访问。
- canonical、Open Graph、robots、sitemap 和 `CNAME` 全部使用根域名。

Windows PowerShell 可使用：

```powershell
Resolve-DnsName clutchreframe.com -Type A
Resolve-DnsName www.clutchreframe.com -Type CNAME
curl.exe -I https://clutchreframe.com/
curl.exe -I https://www.clutchreframe.com/
curl.exe -I https://live-dota2.clutchreframe.com/
```
