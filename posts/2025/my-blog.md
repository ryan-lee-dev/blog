---
date: 2025-12-15
title: Blog 搭建流程
category: blog
tags:
- vitepress
- markdown
- github
description: 最近有点想写blog记录下工作中的收获，因此倒腾了一段时间，选了一个比较简洁的模板来用来写写blog
---

# 搭建过程

## 框架和主题

为了能够使用github的pages进行静态文件的自动编译，采用了如下的框架和主题

- [vitepress](https://vitepress.dev/)
- [vitepress-blog-pure](https://github.com/airene/vitepress-blog-pure)(稍微改了点东西，具体可以看看我的仓库)

```json
{
	"name": "vitepress-blog-pure",
	"version": "1.0.0",
	"description": "",
	"main": "index.ts",
	"scripts": {
		"dev": "vitepress dev --host 127.0.0.1",
		"build": "vitepress build",
		"preview": "vitepress preview"
	},
	"keywords": [],
	"author": "",
	"type": "module",
	"license": "ISC",
	"devDependencies": {
		"vitepress": "2.0.0-alpha.15",
		"globby": "^15.0.0",
		"gray-matter": "^4.0.3",
		"fs-extra": "^11.3.2"
	}
}
```

## 部署流水线

因为平时工作比较忙，所以我想着使用最简单的方法进行博客的搭建，只进行md文件的编写。其他的流程都是用Action流水线部署得了。🤔

```yaml
# 构建 VitePress 站点并将其部署到 GitHub Pages 的示例工作流程
#
name: Deploy VitePress site to Pages

on:
  # 在针对 `main` 分支的推送上运行。如果你
  # 使用 `master` 分支作为默认分支，请将其更改为 `master`
  push:
    branches: [master]

  # 允许你从 Actions 选项卡手动运行此工作流程
  workflow_dispatch:

# 设置 GITHUB_TOKEN 的权限，以允许部署到 GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# 只允许同时进行一次部署，跳过正在运行和最新队列之间的运行队列
# 但是，不要取消正在进行的运行，因为我们希望允许这些生产部署完成
concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  # 构建工作
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v5
        with:
          submodules: true          # 自动初始化并更新子模块
          fetch-depth: 1
      - uses: pnpm/action-setup@v4 # 如果使用 pnpm，请取消此区域注释
        with:
          version: 9
      # - uses: oven-sh/setup-bun@v1 # 如果使用 Bun，请取消注释
      - name: Setup Node
        uses: actions/setup-node@v6
        with:
          node-version: 24
          cache: pnpm
      - name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      # - name: Debug environment
      #   run: |
      #     pwd
      #     ls -la
      #     ls -la .vitepress/theme/components
      #     ls -la themes
      #     node -v
      #     pnpm -v
      - name: Build with VitePress
        run: pnpm run build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: .vitepress/dist

  # 部署工作
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    needs: build
    runs-on: ubuntu-latest
    name: Deploy
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4

```

![image-20251215234910660](imgs/my-blog/image-20251215234910660.png)
