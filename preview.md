# 本地预览：启动 Hexo 服务看效果

面向本仓库 `ooobudooo/ooobudooo.github.io` 的实际配置编写。
适用版本：Hexo 8.1.2、hexo-server 3.x、hexo-theme-keep 4.3.0。
命令都在站点根目录 `D:\WebsiteProj\ObudosZone` 下执行。

---

## 1. 预览服务器是怎么工作的

`hexo server` 会在**内存里**实时渲染站点并起一个本地 HTTP 服务：

- **不写入 `public/`**，所以预览不会弄脏工作区，也不会影响 Git 状态；
- 默认**监听文件变化**，改了 `source/` 里的内容会自动重新生成，刷新浏览器即可看到；
- 只在本机运行，**和线上 GitHub Pages 完全隔离**，预览时看到什么都不会被发布出去。

因此它是改完代码后、提交前的第一道自检。上线流程见 [upload.md](upload.md)。

---

## 2. 最快启动

```powershell
cd D:\WebsiteProj\ObudosZone
npx hexo server
```

看到这行就说明起来了：

```
INFO  Hexo is running at http://localhost:4000/ . Press Ctrl+C to stop.
```

浏览器打开 **<http://localhost:4000/>**（默认端口 4000）。

停止服务：在该终端按 **Ctrl + C**。

也可以用 `package.json` 里预置的脚本，效果一样：

```powershell
npm run server
```

> 两种写法的区别：`npx hexo server` 直接调 CLI；`npm run server` 走脚本。
> 要给脚本传参时，必须用 `--` 分隔，例如 `npm run server -- -p 4001`。

---

## 3. 常用参数

`hexo server` 支持的选项（来自 `hexo server --help`）：

| 参数 | 作用 | 示例 |
| --- | --- | --- |
| `-p, --port` | 指定端口，默认 4000 | `npx hexo server -p 4001` |
| `-o, --open` | 启动后自动用默认浏览器打开 | `npx hexo server -o` |
| `-i, --ip` | 指定监听地址，默认绑定所有网卡 | `npx hexo server -i 127.0.0.1` |
| `-s, --static` | 只当静态服务器用，直接托管 `public/`，不重新构建、不监听变化 | `npx hexo server -s` |
| `-l, --log` | 打开访问日志 | `npx hexo server -l` |

全局参数对 `server` 同样有效：

| 参数 | 作用 |
| --- | --- |
| `--draft` | 把 `source/_drafts/` 里的草稿也渲染出来 |
| `--debug` | 输出详细调试信息，排查渲染异常时用 |
| `--silent` | 只输出错误 |

**常用组合：**

```powershell
# 换端口 + 自动开浏览器
npx hexo server -p 4001 -o

# 连草稿一起预览（推荐日常写作用这个）
npx hexo server --draft

# 只监听本机，不暴露给局域网
npx hexo server -i 127.0.0.1
```

---

## 4. 预览草稿（`source/_drafts/`）

草稿默认**不参与渲染**——这是 `render_drafts: false` 的默认行为，也是它安全的理由：
写到一半的文章不会被误发布。

仓库里现有一篇未完成稿 `source/_drafts/leeze.md`，按下面的方式预览：

```powershell
npx hexo server --draft
```

打开首页就能看到草稿文章，归档页里也会出现。

关于它的访问地址，有一个实测细节：

- 草稿**没有 front matter**（没有 `title` / `date`）时，Hexo 会用**当天日期**生成固定链接，
  例如 `2026-10-22` 预览时地址是 **`/2026/10/22/leeze/`**，而且**每天都会变**；
- 所以写草稿时建议直接带上 front matter，链接才稳定、标题才正常：

```markdown
---
title: 关于北京丽泽商务区城市设计的一点思考
date: 2026-10-21 21:00:00
---
```

**写好之后怎么发布**（二选一）：

```powershell
# 方式一：正式发布为博客文章（移动到 source/_posts/）
npx hexo publish leeze

# 方式二：作为「文章」页的长文（移动到 source/essays/，并手动补目录链接）
Move-Item source\_drafts\leeze.md source\essays\leeze.md
# 然后编辑 source/essays/index.md 增加一行：
# - [标题](/essays/leeze.html)
```

> ⚠️ 默认 `hexo server`（不加 `--draft`）访问草稿地址会返回 404，这是正常的。

---

## 5. 用手机 / 局域网设备预览

`hexo server` 默认绑定所有网卡，所以同一 Wi-Fi 下的手机可以直接访问：

```powershell
# 1) 先查本机局域网 IP
ipconfig | Select-String "IPv4"

# 2) 启动（显式绑定所有网卡更稳）
npx hexo server -i 0.0.0.0 -p 4000
```

然后在手机浏览器打开 `http://<你的IPv4地址>:4000/`，例如 `http://192.168.1.20:4000/`。

打不开时依次检查：

1. 电脑和手机是否在**同一个** Wi-Fi；
2. Windows 防火墙是否拦截了 Node.js（首次运行会弹提示，选「允许专用网络」）；
3. IP 是否填的是 `192.168.x.x` / `10.x.x.x` 这类内网地址，而不是 `127.0.0.1`。

---

## 6. 想验证“真实构建产物”就用 `-s`

日常预览是内存渲染，和最终产物偶尔会有细微差别（比如增量构建缓存）。
想确认 `public/` 里真实生成的结果，用静态模式：

```powershell
npx hexo clean
npx hexo generate
npx hexo server -s     # 直接托管 public/，不重建、不监听
```

这时浏览器看到的就是即将被 `hexo deploy` 推上线的那份东西。
**部署前想最后确认一次，推荐用这个方式。**

---

## 7. 预览环境与线上的差异

预览能覆盖 95% 的情况，但下面这些只有上线后才是真的：

| 项目 | 本地预览 | 线上 |
| --- | --- | --- |
| 地址 | `http://localhost:4000/` | `https://ooobudooo.github.io/` |
| 协议 | HTTP | HTTPS（混合内容会被浏览器拦截） |
| 域名/路径 | 根路径 `/` | 根路径 `/`（本站 `url` 无子目录，二者一致） |
| 草稿 | 加 `--draft` 可见 | 永不可见 |
| 外部资源 | 受本机网络影响 | 受 CDN / 访客网络影响 |
| 构建方式 | 内存、增量 | 全量构建后推送 |
| 缓存 | 本机浏览器缓存 | 浏览器 + GitHub Pages 边缘缓存 |

所以：本地正常≠线上一定正常。涉及**外链、HTTPS 资源、跨域接口**的改动，部署后一定要再线上验一遍。

---

## 8. 常见问题

| 现象 | 原因 | 解决 |
| --- | --- | --- |
| `Port 4000 has been used. Try other port instead.` | 端口被占用（常见于上次没关干净） | `npx hexo server -p 4001`；或先 `Get-NetTCPConnection -LocalPort 4000` 找到进程再结束 |
| 改了文件页面没变化 | 浏览器缓存 / 监听未生效 / 改错目录 | Ctrl+F5 硬刷新；看终端有没有重新生成日志；必要时 Ctrl+C 重启 |
| 改了配置但完全没效果 | `_config.yml` 或 `keep.yml` 改动需要重新加载 | 停掉服务重新 `npx hexo server`；仍不行就 `npx hexo clean` 后重启 |
| 草稿地址 404 | 启动时没加 `--draft` | `npx hexo server --draft` |
| 草稿每天地址都在变 | 草稿缺 front matter，用了当天日期 | 给草稿补 `title` 和 `date` |
| 样式全丢、页面裸奔 | 主题样式没编译 / 缓存残留 | `npx hexo clean` 后重启；确认 `themes/keep` 或 npm 主题存在 |
| 中文显示乱码 | 文件编码不是 UTF-8 | 编辑器另存为 UTF-8（无 BOM） |
| 启动直接报 `FATAL` | 配置 YAML 语法错误 | 看报错指向的文件和行号；YAML 缩进必须用空格、不能用 Tab |
| 首页 404 但其他页正常 | 访问了 `/index.html` 或多余路径 | 直接访问 `http://localhost:4000/` |

---

## 9. 速查表

```powershell
cd D:\WebsiteProj\ObudosZone

npx hexo server                  # 默认 http://localhost:4000/
npx hexo server -p 4001          # 换端口
npx hexo server -o               # 启动并自动打开浏览器
npx hexo server --draft          # 连草稿一起预览
npx hexo server -i 0.0.0.0       # 允许局域网/手机访问
npx hexo server -s               # 只托管 public/（需先 generate）

# 停止：在该终端按 Ctrl + C

# 预览前的“重置键”：构建结果诡异时先清一遍
npx hexo clean
```

预览满意之后，走 [upload.md](upload.md) 完成提交与部署。
