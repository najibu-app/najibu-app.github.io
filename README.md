# My GitHub Pages Blog

A simple and elegant blog powered by Jekyll and GitHub Pages.

## 🌟 Features

- Clean and responsive design using the Minima theme
- Markdown-based blog posts
- Automatic RSS feed generation
- SEO optimization
- Fast loading static site
- Mobile-friendly responsive layout

## 🚀 Quick Start

This site is automatically deployed to GitHub Pages. Simply push changes to the main branch and they'll be live at https://najibu-app.github.io

### Local Development

To run this site locally:

1. **Install Prerequisites**
   ```bash
   # Install Ruby (if not already installed)
   # Install Bundler
   gem install bundler
   ```

2. **Clone and Setup**
   ```bash
   git clone https://github.com/najibu-app/najibu-app.github.io.git
   cd najibu-app.github.io
   bundle install
   ```

3. **Run Locally**
   ```bash
   bundle exec jekyll serve
   ```

4. **View Your Site**
   Open http://localhost:4000 in your browser

## 📝 Writing Posts

Create new blog posts in the `_posts` directory following this naming convention:
```
YYYY-MM-DD-title-of-post.md
```

Each post should start with front matter:
```yaml
---
layout: post
title: "Your Post Title"
date: YYYY-MM-DD HH:MM:SS +TIMEZONE
categories: category1 category2
tags: tag1 tag2
---
```

## 📁 Project Structure

```
najibu-app.github.io/
├── _posts/              # Blog posts
├── _config.yml          # Jekyll configuration
├── index.md             # Homepage
├── about.md             # About page
├── Gemfile              # Ruby dependencies
└── README.md            # This file
```

## 🎨 Customization

- **Theme**: Currently using the Minima theme. You can customize by:
  - Modifying `_config.yml`
  - Adding custom CSS in `assets/css/style.scss`
  - Creating custom layouts in `_layouts/`

- **Configuration**: Edit `_config.yml` to update site title, description, and other settings

## 🔧 Technologies Used

- **Jekyll**: Static site generator
- **GitHub Pages**: Hosting platform
- **Markdown**: Content writing format
- **Liquid**: Templating language
- **SCSS**: Styling (optional)

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

## 🤝 Contributing

Contributions, issues, and feature requests are welcome! Feel free to check the [issues page](https://github.com/najibu-app/najibu-app.github.io/issues).

## 📞 Contact

- **GitHub**: [@najibu-app](https://github.com/najibu-app)
- **Website**: [najibu-app.github.io](https://najibu-app.github.io)

---

⭐ Star this repo if you find it helpful!