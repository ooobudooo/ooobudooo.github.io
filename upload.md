# 本地改动 → 提交 GitHub → 部署上线

面向本仓库 `ooobudooo/ooobudooo.github.io` 的实际配置编写。
适用版本：Hexo 8.1.2、hexo-deployer-git 4.0.0、hexo-theme-keep 4.3.0。
所有命令都在站点根目录 `D:\WebsiteProj\ObudosZone` 下执行。

---

## 0. 先搞清楚两条分支

这个仓库有两个分支，职责完全不同，这是后面所有操作的基础：

| 分支 | 存放内容 | 谁写入 | 能否手动提交 |
| --- | --- | --- | --- |
| `master` | **源码**：`_config.yml`、`source/`、`package.json` 等 | 你执行 `git push` | ✅ 正常提交都往这里 |
| `main` | **构建产物**：`public/` 目录里的 HTML/CSS/JS/图片 | `hexo deploy` 自动推送 | ❌ 不要手动提交 |

`main` 分支是 GitHub Pages 的发布源。`hexo deploy` 每次都会用 `git push --force` **整体覆盖** `main`，
所以你在 `main` 上做的任何手动修改都会在下次部署时丢失。

> 一句话原则：**只改 `master`，`main` 交给 `hexo deploy`。**

---

## 1. 每次开工前的检查

```powershell
cd D:\WebsiteProj\ObudosZone

# 确认工作区干净、看清楚有哪些改动
git status

# 当前所在分支应为 master
git branch --show-current

# 如果上次是在别的机器/网页上改过，先同步远端（有冲突再处理）
git pull --rebase origin master
```

如果 `git status` 里冒出了 `public/`、`node_modules/`、`.deploy_git/`、`db.json` 这些目录，
说明 `.gitignore` 失效了，**先修好再提交**，否则会把几十 MB 的产物和依赖提交进去。

---

## 2. 常见改动的正确位置

先找对文件，比事后返工便宜得多。

| 你想改什么 | 改哪里 | 备注 |
| --- | --- | --- |
| 站点标题、副标题、作者、网址 | `_config.yml` | 与 Hexo 默认值相同的项不用写 |
| 主题外观：主色、首页大图、导航菜单 | `source/_data/keep.yml` | ⚠️ 见下方「浅合并」提醒 |
| 友链内容 | `source/_data/links.yml` | 顶层必须是数组 |
| 相册图片列表 | `source/_data/photos.yml` | 顶层必须是数组 |
| 首页文章（博客体） | `source/_posts/*.md` | 会自动出现在首页、归档、RSS |
| 「文章」页里的长文 | `source/essays/*.md` | ⚠️ 还要手动加链接，见下文 |
| 「文章」页的目录 | `source/essays/index.md` | 手写列表，不是自动生成 |
| 新增独立页面 | `npx hexo new page <名字>` | 生成 `source/<名字>/index.md` |
| 图片等静态资源 | `source/images/` | 引用路径写 `/images/xxx.png` |
| 未完成、不想发布的稿子 | `source/_drafts/` | 默认不渲染，预览要加 `--draft` |

### ⚠️ 关于 `source/_data/keep.yml` 的浅合并

主题在 `generateBefore` 阶段用**按顶层键的浅合并**把这份文件盖到主题默认配置上：

```js
hexo.theme.config = { ...hexo.theme.config, ...data.keep }
```

两条硬约束：

1. 只写「与主题默认值不同」的顶层区块，相同的区块删掉即可；
2. **保留的区块必须把字段写全**——因为是整块替换，漏写的字段会变成 `undefined`，而不是回退默认值。

### ⚠️ 在 `source/essays/` 加文章要两步

1. 新建 `source/essays/<名字>.md`，**必须带 front matter**：

```markdown
---
title: 文章标题
date: 2026-10-22 13:00:00
---
```

2. 打开 `source/essays/index.md`，在列表里补一行：

```markdown
- [文章标题](/essays/<名字>.html)
```

只做第 1 步的话，页面能通过 `/essays/<名字>.html` 直接访问，但目录页里不会出现它。

### ⚠️ 不要随手在 `source/` 里放说明文件

Hexo 会把 `source/` 下所有非 `_` 开头的 `.md` 都当成页面渲染。
放说明文档请放在仓库根目录（如本文件），根目录的 `.md` 不会进站点。

---

## 3. 本地自检（强烈建议，别跳过）

```powershell
# 清掉旧产物和缓存，确保不是拿陈旧结果骗自己
npx hexo clean

# 构建，观察有没有报错
npx hexo generate
```

构建成功会输出类似 `INFO  74 files generated in 380 ms`。

想先看效果再上线，另开一个终端启动预览（详见 [preview.md](preview.md)）：

```powershell
npx hexo server
# 浏览器打开 http://localhost:4000/
```

---

## 4. 提交源码到 GitHub 的 master

```powershell
cd D:\WebsiteProj\ObudosZone

# 1) 先看清楚要提交什么
git status --short

# 2) 暂存全部改动（.gitignore 已排除产物和依赖）
git add -A

# 3) 写清楚的提交信息
git commit -m "feat: 新增文章《xxx》"

# 4) 推送到 GitHub
git push origin master
```

提交信息建议沿用仓库现有风格，一眼能看懂改了什么：

| 前缀 | 用途 | 例子 |
| --- | --- | --- |
| `feat:` | 新功能/新内容 | `feat: 新增友链 MZTLAB` |
| `fix:` | 修 bug | `fix: 修复首页大图不显示` |
| `chore:` | 杂项/配置 | `chore: 更新 keep 主题配置注释` |
| `docs:` | 文档 | `docs: 补充上传与预览说明` |
| `refactor:` | 重构 | `refactor: 精简站点配置` |

推送成功的输出形如：

```
ebc7baf..2b5c26d  master -> master
```

> 💡 推送走的是 SSH（`~/.ssh/config` 把 `github.com` 转到了 `ssh.github.com:443`）。
> 本机 `hosts` 里 `github.com` 被指向 `127.0.0.1`，所以**浏览器/HTTPS 方式访问 GitHub 会失败**，
> 但 `git push` 不受影响。推送前可用 `ssh -T git@github.com` 自检，正常会显示
> `Hi ooobudooo! You've successfully authenticated...`。

---

## 5. 部署到网站

### 方式 A：一条命令（推荐）

```powershell
npx hexo generate --deploy
```

先构建再部署，等价于 `npx hexo g -d`，最不容易出错。

### 方式 B：分两步

```powershell
npx hexo generate
npx hexo deploy
```

### ⚠️ 最容易踩的一个坑

`hexo deploy` **只有在 `public/` 目录不存在时才会自动构建**（见 `hexo/dist/plugins/console/deploy.js`）。
如果 `public/` 已经存在但内容是旧的，`hexo deploy` 会**直接把旧内容推上线**，看起来就像“改了没生效”。

所以：

- 永远先 `hexo generate`，或者直接用 `hexo generate --deploy`；
- 或者 `hexo deploy --generate`（等价）；
- 千万别在改完源码后直接 `npx hexo deploy` 就以为完事了。

### 部署过程发生了什么

1. 把 `public/` 的内容同步进临时仓库 `.deploy_git/`；
2. 提交一条 `Site updated: yyyy-MM-dd HH:mm:ss` 的记录；
3. 执行 `git push -u git@github.com:ooobudooo/ooobudooo.github.io.git HEAD:main --force`；
4. 强推覆盖远端 `main`。

成功的输出形如：

```
[master 2b5c26d] Site updated: 2026-10-22 13:13:08
...
INFO  Deploy done: git
   ebc7baf..2b5c26d  HEAD -> main
```

最后一行出现 `HEAD -> main` 且没有 `error:`，就说明推送成功了。

---

## 6. 验证线上效果

GitHub Pages 收到新提交后需要重建，通常 **30–90 秒**生效。

```powershell
# 直接看返回码
curl.exe -sS -o NUL -w '%{http_code}\n' https://ooobudooo.github.io/

# 或者只取页面标题
$r = Invoke-WebRequest https://ooobudooo.github.io/ -UseBasicParsing
if ($r.Content -match '(?s)<title>(.*?)</title>') { $matches[1].Trim() }
```

浏览器里务必 **Ctrl + F5 强制刷新**，避免看到缓存。

上线自检清单：

- [ ] `git push origin master` 输出末尾有 `master -> master`
- [ ] `git log --oneline -1` 能看到刚提交的记录
- [ ] `hexo generate` 无 `FATAL` / `ERROR`
- [ ] `hexo deploy` 输出末尾有 `HEAD -> main`
- [ ] 线上首页、新文章页、图片地址都能打开（200）
- [ ] 手机上也刷一下，确认不是本机缓存

---

## 7. 从零到上线：速查版

```powershell
cd D:\WebsiteProj\ObudosZone

# 改代码 / 加文章 / 换图片……

npx hexo clean                       # 清产物与缓存
npx hexo generate                    # 本地构建
npx hexo server                      # 另开终端预览，Ctrl+C 退出

git status --short                   # 看清楚改了什么
git add -A
git commit -m "feat: 本次改了什么"
git push origin master               # 源码 → master

npx hexo generate --deploy           # 构建并发布 → main（强制覆盖）

# 等 30–90 秒，Ctrl+F5 打开 https://ooobudooo.github.io
```

---

## 8. 常见问题排查

| 现象 | 原因 | 解决 |
| --- | --- | --- |
| `Could not read from remote repository` | SSH 认证/网络问题 | `ssh -T git@github.com` 自检；确认 `~/.ssh/id_ed25519` 存在且公钥已加到 GitHub |
| `! [rejected] master -> master (non-fast-forward)` | 远端有本地没有的提交 | `git pull --rebase origin master` 后再 `git push` |
| `nothing to commit, working tree clean` | 没有实际改动，或文件被 `.gitignore` 忽略 | `git status --short` 确认；必要时看 `.gitignore` 是否误伤 |
| `git status` 里出现 `public/`、`node_modules/` | `.gitignore` 规则失效 | 检查 `.gitignore`（注释必须独占一行，不能写在规则后面） |
| 部署成功但页面没变 | Pages 还在重建 / 浏览器缓存 / 忘了 `generate` | 等 1–2 分钟 + Ctrl+F5；确认用的是 `generate --deploy` |
| 部署报 `nothing to commit` 或内容为空 | `public/` 没构建 | 先 `npx hexo generate` |
| `hexo deploy` 报 `Deployer not found` | `hexo-deployer-git` 没装 | `npm ci` 重新安装依赖 |
| 页面中文乱码 | 文件不是 UTF-8 | 用 UTF-8 保存 `.md` / `.yml` |
| 构建报 `EPERM` / 权限被拒 | 目录权限异常 | 换到有写权限的目录，或检查 Windows 文件权限 |
| 图片不显示 | 路径写错 | 图片放 `source/images/`，正文用 `/images/xxx.png`（以 `/` 开头） |

---

## 9. 换一台新电脑怎么办

```powershell
# 1) 克隆源码（默认就是 master）
git clone git@github.com:ooobudooo/ooobudooo.github.io.git
cd ooobudooo.github.io

# 2) 按锁文件精确还原依赖
npm ci

# 3) 配好 Git 身份（只需一次）
git config user.name "Obudo"
git config user.email "50094576@qq.com"

# 4) 生成并添加 SSH key（如尚未配置）
ssh-keygen -t ed25519 -C "50094576@qq.com"
# 把 ~/.ssh/id_ed25519.pub 的内容贴到 GitHub → Settings → SSH and GPG keys

# 5) 自检
ssh -T git@github.com
npx hexo generate
```

> 新电脑上如果没有 `themes/keep` 目录，Hexo 会自动使用 npm 依赖里的主题包，
> 效果与本地克隆完全一致（两者是同一版本）。所以不必单独克隆主题。
