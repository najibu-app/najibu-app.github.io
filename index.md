---
layout: home
title: Welcome to My Blog
---

# Welcome to My GitHub Pages Blog! 🎉

Hello and welcome to my personal blog powered by Jekyll and GitHub Pages!

## About This Blog

This is where I share my thoughts, experiences, and discoveries in the world of technology, programming, and beyond. Whether you're a fellow developer, a curious learner, or just someone who stumbled upon this site, I hope you'll find something interesting here.

## What You'll Find Here

- **Technical tutorials** and how-to guides
- **Project showcases** and development stories
- **Learning experiences** and lessons learned
- **Tips and tricks** for developers
- **Personal reflections** on technology and life

## Latest Posts

Here are my most recent blog posts:

{% for post in site.posts limit:5 %}
- [{{ post.title }}]({{ post.url }}) - *{{ post.date | date: "%B %d, %Y" }}*
{% endfor %}

## Get In Touch

I love connecting with fellow developers and tech enthusiasts! Feel free to:

- Check out my projects on [GitHub](https://github.com/najibu-app)
- Read all my [blog posts](/posts/)
- Follow my coding journey

## Subscribe

Want to stay updated with my latest posts? You can subscribe to my [RSS feed](/feed.xml) to never miss an update!

---

Thanks for visiting, and happy reading! 📚✨