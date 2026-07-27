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

1. 打开 GitHub 组织设置。
2. 进入 **Pages**。
3. 添加 `clutchreframe.com`。
4. 按 GitHub 提示在 Cloudflare 添加 TXT 验证记录。
5. 等待 GitHub 显示域名验证成功。

不要删除 GitHub 要求保留的 TXT 验证记录。

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

### 6.1 根域名

为 `clutchreframe.com` 配置以下四条 A 记录：

| Type | Name | IPv4 address | Proxy status |
| --- | --- | --- | --- |
| A | `@` | `185.199.108.153` | DNS only |
| A | `@` | `185.199.109.153` | DNS only |
| A | `@` | `185.199.110.153` | DNS only |
| A | `@` | `185.199.111.153` | DNS only |

可同时配置 GitHub Pages 官方列出的四条 AAAA 记录：

```text
2606:50c0:8000::153
2606:50c0:8001::153
2606:50c0:8002::153
2606:50c0:8003::153
```

### 6.2 www 兼容入口

添加：

| Type | Name | Target | Proxy status |
| --- | --- | --- | --- |
| CNAME | `www` | `clutchreframe.github.io` | DNS only |

当 Pages Custom domain 使用 `clutchreframe.com` 且两组 DNS 都正确时，
GitHub Pages 会将 `www.clutchreframe.com` 自动重定向到根域名。

### 6.3 不得修改的记录

不要修改或删除现有子域名记录，包括但不限于：

```text
clips.clutchreframe.com
live.clutchreframe.com
live-dota2.clutchreframe.com
support.clutchreframe.com
```

不要添加 `*.clutchreframe.com` 通配符记录，也不要改动邮件相关 MX、SPF、
DKIM 或 DMARC 记录。

## 7. HTTPS

等待 GitHub Pages 显示 DNS check successful 并完成证书签发，然后启用：

```text
Enforce HTTPS
```

DNS 和证书可能需要数分钟到 24 小时。等待期间不要反复删除 Custom
domain 或 DNS 记录。

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
