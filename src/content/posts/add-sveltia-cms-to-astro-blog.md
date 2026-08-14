---
title: 给 Astro 静态博客加一个可以在线写文章的后台
published: 2026-08-14
description: 使用 Sveltia CMS 和 GitHub，把静态博客改造成可以在浏览器中新增、编辑和发布文章的内容系统。
image: api
tags: [Astro, Sveltia CMS, GitHub, 博客]
category: 技术
draft: false
---

静态博客速度快、维护成本低，但每次写文章都要在本地打开编辑器、提交 Git，再等待构建。对于日常记录来说，这套流程多少有些繁琐。

我的博客使用 Astro，文章以 Markdown 保存在 GitHub。为了能直接在浏览器里写文章，我给它接入了 Sveltia CMS。它不需要额外数据库，保存文章的动作本质上仍然是向 GitHub 仓库提交 Markdown 文件。

## 最终效果

后台地址位于：

```text
https://blog.voqz.de/admin/
```

登录后可以完成这些操作：

- 新增、修改和删除文章
- 设置发布日期、标签、分类和草稿状态
- 上传文章封面
- 编辑关于页、友情链接和留言板
- 保存后自动触发 GitHub Actions 部署

## 一、添加后台入口

Astro 会把 `public` 目录中的文件原样复制到构建产物，因此后台入口放在 `public/admin/index.html`：

```html
<!doctype html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="robots" content="noindex, nofollow" />
    <title>内容后台</title>
  </head>
  <body>
    <script src="https://unpkg.com/@sveltia/cms@0.189.1/dist/sveltia-cms.js"></script>
  </body>
</html>
```

我固定了 CMS 版本，避免上游更新后后台行为突然发生变化。

## 二、连接 GitHub 仓库

在 `public/admin/config.yml` 中配置 GitHub 后端：

```yaml
backend:
  name: github
  repo: 1UmU/my-blog
  branch: master

site_url: https://blog.voqz.de
publish_mode: simple
```

文章目录必须使用相对于仓库根目录的路径：

```yaml
collections:
  - name: posts
    label: 文章
    folder: src/content/posts
    create: true
    extension: md
    format: frontmatter
```

如果误写成系统绝对路径，后台可能无法正确读取仓库内容。

## 三、让字段匹配 Astro Content Schema

CMS 表单字段必须和 Astro 的内容结构一致。例如：

```yaml
fields:
  - { label: 标题, name: title, widget: string }
  - label: 发布日期
    name: published
    widget: datetime
    type: date
    format: YYYY-MM-DD
  - { label: 标签, name: tags, widget: list, required: false }
  - { label: 草稿, name: draft, widget: boolean, default: false }
  - { label: 正文, name: body, widget: markdown }
```

我还启用了空字段省略：

```yaml
output:
  omit_empty_optional_fields: true
```

这样可选的更新日期为空时，不会生成 `updated: null`，可以减少内容校验失败的可能。

## 四、配置图片上传

```yaml
media_folder: public/assets/images/posts
public_folder: /assets/images/posts
```

前者是图片在 GitHub 仓库中的保存位置，后者是浏览器访问图片时使用的路径。两者含义不同，不能混用。

Sveltia CMS 还可以在上传时转换图片：

```yaml
media_libraries:
  default:
    config:
      max_file_size: 5242880
      transformations:
        raster_image:
          format: webp
          quality: 85
          width: 1920
```

这样能控制图片体积，避免一张原图拖慢整篇文章。

## 五、登录方式与权限

个人博客可以先使用 GitHub Fine-grained personal access token 登录。Token 只授权当前博客仓库，并只开放：

```text
Contents: Read and write
```

Token 不应该写入配置、提交到仓库或发给其他人。更进一步，可以部署 OAuth 服务，实现标准的 GitHub 授权登录。

## 六、验证构建

发布前至少执行：

```bash
pnpm check
pnpm build
```

并确认构建目录中存在：

```text
dist/admin/index.html
dist/admin/config.yml
```

## 写在最后

接入 CMS 后，博客仍然是一个纯静态网站，也没有增加需要长期维护的数据库。变化只在写作入口：以前必须在本地修改文件，现在打开浏览器就能完成编辑和发布。对个人博客来说，这是一种很轻量的平衡。
