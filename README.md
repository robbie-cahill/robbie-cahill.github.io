# Jekyll GitHub Pages Site

A simple, clean Jekyll site hosted on GitHub Pages. This site demonstrates how to create a professional-looking website using Jekyll's static site generation capabilities.

## Features

- 📝 **Markdown Content**: Write content in simple Markdown syntax
- 🎨 **Minima Theme**: Clean, responsive design out of the box
- 📱 **Mobile Friendly**: Responsive layout that works on all devices
- 🔍 **SEO Optimized**: Built-in SEO tags and sitemap generation
- 📰 **Blog Ready**: Full blogging functionality with posts, categories, and tags
- ⚡ **Fast Loading**: Static site generation for optimal performance

## Quick Start

### Local Development

1. **Install Dependencies**
   ```bash
   bundle install
   ```

2. **Serve the Site Locally**
   ```bash
   bundle exec jekyll serve
   ```

3. **View Your Site**
   Open your browser to `http://localhost:4000`

### GitHub Pages Deployment

1. **Enable GitHub Pages**
   - Go to your repository settings
   - Scroll down to "Pages" section
   - Select "Deploy from a branch"
   - Choose "main" branch and "/ (root)" folder
   - Click "Save"

2. **Your Site Will Be Available At**
   ```
   https://[username].github.io/[repository-name]
   ```

## File Structure

```
├── _config.yml          # Site configuration
├── _posts/              # Blog posts directory
│   └── 2025-06-26-welcome-to-jekyll.md
├── _layouts/            # Custom layouts (optional)
├── _includes/           # Reusable components (optional)
├── _sass/               # Sass stylesheets (optional)
├── assets/              # Images, CSS, JS files
├── Gemfile              # Ruby dependencies
├── index.md             # Homepage
├── about.md             # About page
└── README.md            # This file
```

## Adding Content

### Creating Pages

Create new `.md` files in the root directory with front matter:

```markdown
---
layout: page
title: Your Page Title
permalink: /your-page/
---

# Your content here
```

### Writing Blog Posts

Add new posts to the `_posts` directory with the naming convention:
`YYYY-MM-DD-title.md`

```markdown
---
layout: post
title: "Your Post Title"
date: 2025-06-26 12:00:00 +1000
categories: category1 category2
tags: [tag1, tag2, tag3]
---

Your post content here...
```

## Customization

### Site Configuration

Edit `_config.yml` to customize:
- Site title and description
- Author information
- Social media links
- Navigation menu
- Theme settings

### Styling

The site uses the Minima theme. You can customize:
- Colors and fonts in `_sass/minima.scss`
- Add custom CSS in `assets/css/style.scss`
- Override theme layouts in `_layouts/`

### Plugins

Current plugins included:
- `jekyll-feed` - RSS feed generation
- `jekyll-sitemap` - XML sitemap
- `jekyll-seo-tag` - SEO meta tags

## Deployment

### GitHub Pages (Recommended)

GitHub Pages automatically builds and deploys your site when you push to the main branch. No additional configuration needed.

### Manual Deployment

1. Build the site:
   ```bash
   bundle exec jekyll build
   ```

2. Deploy the `_site` directory to your web server.

## Troubleshooting

### Common Issues

1. **Site not updating**: Check GitHub Actions tab for build errors
2. **Local development issues**: Run `bundle update` to update dependencies
3. **Theme not loading**: Ensure `theme: minima` is in `_config.yml`

### Useful Commands

```bash
# Install dependencies
bundle install

# Update dependencies
bundle update

# Serve locally with live reload
bundle exec jekyll serve --livereload

# Build for production
bundle exec jekyll build

# Check for issues
bundle exec jekyll doctor
```

## Resources

- [Jekyll Documentation](https://jekyllrb.com/docs/)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [Minima Theme](https://github.com/jekyll/minima)
- [Markdown Guide](https://www.markdownguide.org/)

## License

This project is open source and available under the [MIT License](LICENSE).