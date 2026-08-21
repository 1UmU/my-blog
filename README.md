# UmU Blog

UmU 的个人博客，用于记录技术笔记、项目折腾过程和生活随想。

[![Astro](https://img.shields.io/badge/Astro-5.16.8-BC52EE?logo=astro&logoColor=white)](https://astro.build/)
[![Sveltia CMS](https://img.shields.io/badge/CMS-Sveltia-2F855A)](https://github.com/sveltia/sveltia-cms)
[![Cloudflare Pages](https://img.shields.io/badge/Deploy-Cloudflare%20Pages-F38020?logo=cloudflare&logoColor=white)](https://pages.cloudflare.com/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)

## 站点入口

- 博客首页：<https://blog.voqz.de>
- 内容后台：<https://blog.voqz.de/admin/>
- 游戏中心：<https://blog.voqz.de/games/>
- 扫雷游戏：<https://game.voqz.de>
- GitHub 仓库：<https://github.com/1UmU/my-blog>

## 项目功能

- 基于 Astro、Svelte 和 Tailwind CSS 的静态博客
- 响应式页面、亮暗色模式和可切换文章布局
- Pagefind 全文搜索、RSS、站点地图和 Open Graph 支持
- Markdown/MDX、数学公式、代码高亮和文章目录
- Sveltia CMS 可视化内容管理后台
- 使用 GitHub OAuth 登录后台，无需手动填写 Personal Access Token
- 后台支持文章、图片、站点外观、独立页面和音乐歌单管理
- GitHub 提交后由 Cloudflare Pages 自动构建和发布
- 博客导航中集成游戏中心与管理后台入口

## 发布流程

```text
Sveltia CMS 编辑内容
        |
        v
提交到 GitHub master
        |
        v
Cloudflare Pages 自动构建
        |
        v
发布到 blog.voqz.de
```

后台保存、修改或删除内容后，一般需要等待 1-3 分钟完成线上更新。浏览器仍显示旧内容时，可使用强制刷新或无痕窗口检查。

## 分支约定

| 分支 | 用途 |
| --- | --- |
| `master` | 生产分支，Cloudflare Pages 从该分支发布正式站点 |
| `develop` | 开发与测试分支，功能验证完成后再合并到 `master` |

日常开发默认在 `develop` 进行，不直接修改 `master`。后台内容管理目前会直接提交到 `master`，开始开发前应先同步最新生产内容。

## 技术栈

- [Astro](https://astro.build/) 5
- [Svelte](https://svelte.dev/) 5
- [Tailwind CSS](https://tailwindcss.com/) 3
- [Sveltia CMS](https://github.com/sveltia/sveltia-cms)
- [Pagefind](https://pagefind.app/)
- [Cloudflare Pages](https://pages.cloudflare.com/)
- [Cloudflare Workers](https://workers.cloudflare.com/) OAuth 认证服务

## 本地开发

### 环境要求

- Node.js 22 或更高版本
- pnpm 9.14.4

### 启动项目

```bash
git clone https://github.com/1UmU/my-blog.git
cd my-blog
git switch develop
corepack enable
corepack pnpm install --frozen-lockfile
corepack pnpm dev
```

本地开发地址：<http://localhost:4321>

### 常用命令

| 命令 | 作用 |
| --- | --- |
| `corepack pnpm dev` | 启动本地开发服务器 |
| `corepack pnpm check` | 运行 Astro 类型和内容检查 |
| `corepack pnpm build` | 构建生产站点并生成搜索索引 |
| `corepack pnpm preview` | 本地预览生产构建 |
| `corepack pnpm new-post <名称>` | 创建新的文章文件 |
| `corepack pnpm format` | 使用 Biome 格式化 `src` 目录 |

## Cloudflare Pages 配置

```text
Production branch: master
Build command: pnpm run build
Build output directory: dist
Root directory: 留空
Framework preset: Astro
```

Cloudflare GitHub App 只需要访问 `1UmU/my-blog`。任何 OAuth Client Secret、Cloudflare Token 或其他密钥都不能提交到仓库。

## 目录结构

```text
public/admin/          Sveltia CMS 后台页面与配置
public/assets/         图片、音乐和其他静态资源
src/components/       页面组件
src/config/           站点、导航、外观和音乐配置
src/content/posts/    博客文章
src/content/spec/     关于、友链、留言板等独立页面
src/pages/            Astro 页面与接口
```

## 内容管理

访问 <https://blog.voqz.de/admin/> 后使用 GitHub 登录。完成授权后可以：

- 新建、编辑、发布和删除文章
- 上传文章图片及站点图片
- 修改头像、Logo、桌面和手机壁纸
- 管理音乐文件、封面、歌词和播放列表
- 编辑关于、友情链接和留言板页面

后台操作本质上会修改 GitHub 仓库，因此登录用户必须拥有 `1UmU/my-blog` 的写入权限。

## 致谢

本项目基于 [CuteLeaf/Firefly](https://github.com/CuteLeaf/Firefly) 定制，Firefly 基于 [saicaca/fuwari](https://github.com/saicaca/fuwari) 开发。感谢原作者和相关开源项目的工作。

主题所使用的第三方素材、图标和组件版权归各自作者或权利方所有。

## 许可证

项目代码遵循 [MIT License](./LICENSE)。使用和分发时请保留原项目要求的版权与许可证声明。
