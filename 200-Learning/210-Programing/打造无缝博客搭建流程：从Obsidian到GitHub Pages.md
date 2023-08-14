---
title: "打造无缝博客搭建流程：从Obsidian到GitHub Pages"
description: "本文将介绍如何使用Obsidian创作博客，在GitHub自动同步，通过GitHub Workflow实现分支合并，结合Astro前端框架搭建网站，自动部署至GitHub Pages。通过这无缝流程，创作者可以专注于内容，无需繁琐操作，打造个人博客。"
pubDate: "2023-08-12 22:43"
heroImage: "https://images.unsplash.com/photo-1504805572947-34fad45aed93?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1200&q=80"
date created: "2023-08-12 22:43"
date modified: "2023-08-12"
tags:
    - writings
---

# 打造无缝博客搭建流程：从Obsidian到GitHub Pages

> True knowledge exists in knowing that you know nothing.
> — <cite>Isocrates</cite>

:::tip
以下内容通过[ChatGPT生成](https://chat.openai.com/share/86232a67-ea2d-4975-8989-a620d6c11b26)+微调（懒得写了😄）
:::

[博客文章仓库](https://github.com/fw6/obsidian-blogs)
[博客站点仓库](https://github.com/fw6/blog)

随着数字时代的到来，个人博客成为展示自我、分享知识的理想平台。在本文中，我将介绍我是如何搭建我的个人博客网站，通过将Obsidian、GitHub以及Astro前端框架等工具有机结合，实现了博客文章的创作、同步和展示的无缝流程。

## 使用Obsidian进行博客创作

我选择使用Obsidian作为博客文章的创作工具，因为它提供了优雅的笔记管理和编辑体验。在Obsidian中，我可以轻松地组织我的思维，编写文章，并添加标签、链接等元素。这使得博客内容的创作变得高效而有趣。

## 实现博客内容自动同步

为了将我在Obsidian中创作的内容自动同步到我的GitHub仓库中，我使用了`obsidian-git`插件。通过配置这个插件，每当我在Obsidian中保存一篇文章，它会自动将文章同步到GitHub仓库的`master`分支中。

## 利用GitHub Workflow实现自动分支合并

在GitHub仓库中，我设置了一个GitHub Workflow，以便在每次将内容推送到`master`分支时自动触发分支合并操作。具体而言，我将`master`分支合并到一个专门用于博客内容的`blog`分支中。这样，我可以确保博客文章的内容始终保持同步，且能够轻松地进行版本控制。

如果自动同步失败☹️，会自动创建PR（将`master`合并到`blog`分支）📢

```yaml
name: CI_auto_merge
on:
  push:
    branches:
      - master
    paths:
      - '**.md'
      - '**.mdx'

jobs:
  auto-merge-blog-branch:
    name: Auto merge blog branch
    runs-on: ubuntu-latest
    permissions: write-all
    steps:
      - name: checkout
        uses: actions/checkout@v3
        with:
          ref: blog

      - name: Set Git config
        run: |
            git config --local user.email "actions@github.com"
            git config --local user.name "Github Actions"

      - name: Merge master back to blog
        run: |
            git fetch --unshallow
            git pull
            git merge --no-ff origin/master -m "Auto-merge master back to blog"
            git push

      - name: Generate a pr after after previous step failed
        if: ${{ failure() }}
        run: |
            gh pr create --base astro --head master --title "Auto Merge Astro Branch" --body "Auto Merge Astro Branch" --label "automerge"
        env:
            GH_TOKEN: ${{ github.token }}

      - name: Trigger Astro CI workflow on master branch at blog repo from outside
        # https://stackoverflow.com/questions/71499150/how-to-run-a-workflow-using-github-cli
        run: gh workflow run astro.yml --ref master --repo git@github.com:fw6/blog.git
        env:
            GH_TOKEN: ${{ secrets.workflow_triggers }}
        # run: |
        #     REPO_OWNER=fw6
        #     REPO_NAME=blog
        #     BRANCH=master
        #     WORKFLOW_ID=astro.yml
        #     ACCESS_TOKEN=${{ secrets.workflow_triggers }}
        #     URL="https://api.github.com/repos/$REPO_OWNER/$REPO_NAME/actions/workflows/$WORKFLOW_ID/dispatches"
        #     PAYLOAD='{"ref":"$BRANCH"}'

        #     curl -X POST -H "Accept: application/vnd.github.v3+json" \
        #         -H "Authorization: Bearer $ACCESS_TOKEN" \
        #         -H "Content-Type: application/json" \
        #         -d "$PAYLOAD" \
        #         "$URL"
```

## 基于Astro前端框架搭建博客网站

为了将博客内容呈现给读者，我选择了Astro前端框架搭建博客网站。我创建了一个独立的代码仓库，并使用Git的`submodule`功能将其与博客文章的`blog`分支关联起来。这样，我可以在网站中展示最新的博客内容，与Obsidian中的文章保持同步。

## 构建GitHub Workflow实现自动部署

在博客网站的源码仓库中，我设置了另一个GitHub Workflow，以实现自动部署。这个Workflow包括以下主要步骤：

1. 同步`submodule`：首先，Workflow会同步博客文章所在仓库的`blog`分支，确保网站源码与最新的博客内容一致。
2. 安装Node.js和依赖项：接下来，Workflow会安装所需的Node.js和依赖项，以准备进行网站的编译和构建。
3. 触发Astro编译脚本：Workflow会执行Astro编译脚本，将博客网站的源码转换为静态文件，准备部署到GitHub Pages上。
4. 部署到GitHub Pages：最后，Workflow将编译后的静态文件部署到GitHub Pages上，使得我的博客内容能够在互联网上访问。

```yaml
# Sample workflow for building and deploying an Astro site to GitHub Pages
#
# To get started with Astro see: https://docs.astro.build/en/getting-started/
#
name: Deploy Astro site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["master"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: true

      # https://github.com/actions/checkout/issues/417#issuecomment-1640448307
      - name: Update submodules from origin master
        env:
            PAT: ${{ secrets.PAT }}
        run: |
            git submodule foreach --recursive git checkout master
            git submodule foreach --recursive git pull origin master

      - name: Remove obsidion directories
        run:
            rm -rf src/content/blog/{_github,_obsidian,_cedict_ts.u8,.obsidian,.github,.gitignore,renovate.json,cedict_ts.u8}

      - name: Setup pnpm
        # You may pin to the exact commit or the version.
        # uses: pnpm/action-setup@c3b53f6a16e57305370b4ae5a540c2077a1d50dd
        uses: pnpm/action-setup@v2.2.4
        with:
            run_install: false
            version: 8

      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version-file: "package.json"
          cache: "pnpm"

      - name: Get pnpm store directory
        id: pnpm-cache
        shell: bash
        run: |
            echo "STORE_PATH=$(pnpm store path)" >> $GITHUB_OUTPUT

      - uses: actions/cache@v3
        name: Setup pnpm cache
        with:
            path: ${{ steps.pnpm-cache.outputs.STORE_PATH }}
            key: ${{ runner.os }}-pnpm-store-${{ hashFiles('**/pnpm-lock.yaml') }}
            restore-keys: |
                ${{ runner.os }}-pnpm-store-

      - name: Install dependencies
        run: pnpm i

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3

      - name: Build with Astro
        run: |
          pnpm build \
            --site "${{ steps.pages.outputs.origin }}" \
            --base "${{ steps.pages.outputs.base_path }}"


      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: "./dist"

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
        uses: actions/deploy-pages@v2
```

通过这个精心设计的GitHub Workflow，我可以实现博客内容的自动更新和部署，让读者随时随地都能够浏览到最新的文章和信息。

## 总结

通过将Obsidian、GitHub和Astro前端框架等工具融合在一起，我成功地搭建了一个高效、自动化的博客网站搭建流程。从创作到同步再到部署，整个流程无缝衔接，让我能够专注于内容创作，而不必过多担心技术细节。希望本文能够为有类似需求的朋友提供一些有用的思路和方法。
