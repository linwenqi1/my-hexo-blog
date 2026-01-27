# My Hexo Blog

## 📖 About

A personal blog built with [Hexo](https://hexo.io/) and [NexT](https://theme-next.js.org/) theme, focusing on technology sharing and life recording.

- **Live Site**: https://www.sky-learner.top/
- **Author**: Lin Wenqi
- **Theme**: NexT (Gemini Scheme)

## ✨ Features

- 🎨 Clean and modern design with NexT Gemini theme
- 📱 Fully responsive layout
- 🔍 Local search functionality
- 💬 Comment system powered by Twikoo
- 📊 Reading progress bar
- ⏱️ Reading time and word count statistics
- 🔗 Social media integration
- 📝 Syntax highlighting with highlight.js
- 📰 RSS feed support
- 🗺️ SEO-friendly with sitemap generation
- 🚀 Fast deployment to GitHub Pages

## 🛠️ Tech Stack

- **Static Site Generator**: Hexo 8.0+
- **Theme**: NexT 8.27+
- **Markdown Renderer**: hexo-renderer-pandoc
- **Deployment**: GitHub Pages
- **Comment System**: Twikoo
- **Analytics**: Optional (Google Analytics, Baidu Analytics)

## 📦 Installation

### Prerequisites

- Node.js (v14.0.0 or higher)
- Git
- Pandoc (for markdown rendering)

### Quick Start

1. **Clone the repository**

   ```bash
   git clone git@github.com:linwenqi1/my-hexo-blog
   cd my-hexo-blog
   ```

2. **Install dependencies**

   ```bash
   npm install
   ```

3. **Install Pandoc** (if not already installed)

   - **macOS**: `brew install pandoc`
   - **Ubuntu/Debian**: `sudo apt install pandoc`
   - **Windows**: Download from [Pandoc releases](https://github.com/jgm/pandoc/releases)

4. **Run local server**

   ```bash
   npm run server
   ```

   Visit `http://localhost:4000` to preview your blog.

## 🚀 Usage

### Create a New Post

```bash
hexo new "My New Post"
```

This will create a new markdown file in `source/_posts/`.

### Generate Static Files

```bash
npm run build
# or
hexo generate
```

### Deploy to GitHub Pages

```bash
npm run deploy
# or
hexo deploy
```

### Clean Cache

```bash
npm run clean
# or
hexo clean
```

## ⚙️ Configuration

### Site Configuration

Edit `_config.yml` in the root directory:

```yaml
# Site Information
title: My Blog
subtitle: 'Recording Technology and Life'
description: 'A blog focused on technology sharing'
author: Lin Wenqi
language: zh-CN
timezone: Asia/Shanghai

# URL
url: https://www.sky-learner.top/
```

### Theme Configuration

The NexT theme configuration is in the first document you provided. Key settings include:

- **Scheme**: Gemini (clean and modern)
- **Sidebar**: Left position, shown on post pages
- **Code Highlighting**: highlight.js with GitHub Dark theme
- **Search**: Local search enabled
- **Comments**: Twikoo integration
- **Social Links**: GitHub, Email

### Customization

To customize the theme:

1. **Avatar**: Place your avatar image at `source/images/avatar.jpg`
2. **Custom Styles**: Edit `source/_data/styles.styl`
3. **Custom Scripts**: Edit `source/_data/body-end.njk`

## 📝 Writing Posts

### Front Matter

Each post should include front matter:

```markdown
---
title: Post Title
date: 2024-01-27 10:00:00
tags:
  - Technology
  - Programming
categories:
  - Tech
---

Your content here...
```

### Using Images

With `post_asset_folder: true`, create a folder with the same name as your post:

```
source/_posts/
  ├── my-post.md
  └── my-post/
      └── image.jpg
```

Reference images in your post:

```markdown
![Image description](image.jpg)
```

## 🔌 Plugins

This blog uses the following Hexo plugins:

- **hexo-deployer-git**: Deploy to GitHub Pages
- **hexo-generator-feed**: Generate RSS feed
- **hexo-generator-sitemap**: Generate sitemap
- **hexo-generator-searchdb**: Enable local search
- **hexo-symbols-count-time**: Show reading time and word count
- **hexo-next-twikoo**: Twikoo comment integration
- **hexo-renderer-pandoc**: Enhanced markdown rendering

## 📄 License

This project is licensed under the **GNU Affero General Public License v3.0 (AGPL-3.0)**. See the [LICENSE](https://www.google.com/search?q=./LICENSE) file for the full text.

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

## 📧 Contact

- **Email**: linwenqi05@outlook.com
- **GitHub**: [@linwenqi1](https://github.com/linwenqi1)

## 🙏 Acknowledgments & Credits

This project incorporates and builds upon the following open-source software:

- **[Hexo](https://github.com/hexojs/hexo)**: Licensed under the **MIT License**. Copyright (c) 2012-2026 Tommy Chen.
- **[NexT Theme](https://github.com/next-theme/hexo-theme-next)**: Licensed under the **AGPL-3.0 License**. Copyright (c) 2017-2026 NexT Team.