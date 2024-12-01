---
title: Github Pages 和 Hexo 搭建博客
---

## 创建仓库

1. 访问：[https://github.com/new](https://github.com/new)
2. 创建仓库，名称写 `xxx.github.io`。
3. 在 GitHub Pages 中找到我们主页的地址为 `https://xxx.github.io/`。

---

## 环境配置

### 1. 下载安装 Git 环境
设置 GitHub 和 SSH Key。

### 2. 搭建 Node.js 环境
推荐使用 nvm：[文章链接](https://cloud.tencent.com/developer/article/1812323)。

---

## Hexo 的使用

### 1. 安装 Hexo
```bash
npm install -g hexo-cli
```

### 2. 初始化 Hexo 框架
```bash
hexo init blog
```

### 3. 安装相关依赖
```bash
npm install
```

### 4. 启动本地服务
```bash
hexo server
```
浏览器访问：[localhost:4000](http://localhost:4000)。

---

## 配置 Next 主题

1. 进入上一步创建的 `blog` 文件夹，将 Next 主题相关文件从 GitHub 克隆到 `themes` 文件夹中：
   ```bash
   git clone https://github.com/iissnan/hexo-theme-next themes/next
   ```

2. 配置 Hexo 的主题参数：
   - 修改根目录的 `_config.yml` 文件中的 `theme` 参数为：
     ```yaml
     theme: next
     ```

3. 启动 Hexo 服务，验证 Next 主题是否启用。

---

## 浏览器访问 localhost:4000 时出错

### 错误内容
浏览器显示内容如下所示：
```html
{% extends '_layout.swig' %} {% import '_macro/post.swig' as post_template %} {% import '_macro/sidebar.swig' as 
sidebar_template %} {% block title %}{{ config.title }}{% if theme.index_with_subtitle and config.subtitle %} - 
{{config.subtitle }}{% endif %}{% endblock %} {%  block page_class %} {% if is_home() %}page-home{% endif -%} {% 
endblock %} {% block content %} {% for post in page.posts %} {{ post_template.render(post, true) }} {% endfor %} {% 
include '_partials/pagination.swig' %} {% endblock %} {% block sidebar %} {{ sidebar_template.render(false) }} {% 
endblock %}
```

### 解决方案
这是因为 Hexo 在 5.0 之后删除了 `swig`，需要自己手动安装：
```bash
npm i hexo-renderer-swig
```

---

## 添加博客内容

1. 将写好的 Markdown 放到 `source/_posts` 目录。
2. 将相应的图片放到 `source/images` 目录。
3. 启动 Hexo 服务（如果之前没有关闭，请忽略此步骤）。
4. 通过浏览器访问博客主页，可以发现更改已生效。

---

## 将生成的静态页面部署到 GitHub 上

### 1. 安装 `deploy-git`
```bash
npm install hexo-deployer-git --save
```

### 2. 修改站点配置文件 `_config.yml` 的最后部分：
```yaml
deploy:
  type: git
  repo: https://github.com/YourgithubName/YourgithubName.github.io.git
  branch: master
```

### 3. 部署博客
```bash
hexo clean       # 清除之前生成的内容
hexo generate    # 生成静态文章，缩写 hexo g
hexo deploy      # 部署文章，缩写 hexo d
```

现在，你可以在 [http://xxx.github.io](http://xxx.github.io) 看到你的博客了！