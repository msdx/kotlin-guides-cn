---
layout: page
title: 贡献
site_nav_category: contribute
site_nav_category_order: 400
is_site_nav_category: true
---

我们非常欢迎和支持为这个网站作出贡献！

如果想为本网站做贡献，小的修改可以直接创建 Pull Request；对于更大的贡献，我们建议先在 [issue tracker](https://github.com/android/kotlin-guides/issues)上创建一个 issue。

Pull Request 应该针对 [GitHub 仓库](https://github.com/android/kotlin-guides)中的 `master` 分支。每隔几周我们就会更新[修改日志](changelog.html)将被更新，并且该时间段内的所有更改发布到 `gh-pages` 分支。

**我们期待大家的所有贡献！**


## 开发

在改动过程中，你可以在本机上运行本站点。

### 安装 Ruby 和 Bundler

确保你已经安装了 Ruby 和 [Bundler](http://bundler.io/)。

    gem install bundler

### 一次性设置

    bundle install --path vendor/bundle

_注意：如果你使用的是 Mac OS 并且无法安装 nokogiri，请先运行 `brew unlink xz`，然后再安装，最后再运行`brew link xz`。_

### 运行网站

    bundle exec jekyll serve

在你的浏览器中打开 [http://127.0.0.1:4000/kotlin-guides/](http://127.0.0.1:4000/kotlin-guides/)。
