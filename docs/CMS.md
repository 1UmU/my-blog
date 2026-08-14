# 在线内容后台

博客使用 Sveltia CMS 管理 GitHub 仓库中的 Markdown 内容。后台不会保存 GitHub
密钥到仓库，也不需要额外数据库。

## 入口

部署完成后访问：

```text
https://blog.voqz.de/admin/
```

## 首次登录

1. 点击 `Sign in with Token`。
2. 按页面链接在 GitHub 创建 Fine-grained personal access token。
3. Repository access 只选择 `1UmU/my-blog`。
4. Repository permissions 设置 `Contents: Read and write`，其余保持只读或不授权。
5. 设置较短的过期时间，例如 30 或 90 天。
6. 将 Token 粘贴到后台登录框。

Token 只保存在当前浏览器的本地存储中。不要在公共电脑登录，不要将 Token 放进
`config.yml`、GitHub 仓库、聊天或截图。Token 泄露后应立即在 GitHub 撤销。

## 发布流程

后台保存会直接提交到 `master` 分支。现有 GitHub Actions 随后构建博客，并将结果
发布到 `pages` 分支。正常情况下数十秒到数分钟后线上生效。

- `draft: true`：保存到仓库，但不在博客公开展示。
- `draft: false`：参与下一次构建并公开展示。
- 图片上传到 `public/assets/images/posts/`，CMS 会转换为 WebP 并限制为 5 MB。

可以在 GitHub 仓库的 `Actions` 页面查看构建状态。发布失败时，先打开失败的工作流
查看 `Build site` 步骤日志。

## 后续升级 OAuth

当前适合单人使用 Token 登录。需要更方便的一键登录时，可部署官方
`sveltia/sveltia-cms-auth` Cloudflare Worker，并在 `backend.base_url` 中填写 Worker
地址。GitHub OAuth Client Secret 只能保存在 Worker Secret 中，不能进入博客仓库。
