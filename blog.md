---
layout: page
title: Blog
permalink: /blog/
in_headers: true
---

<div class="home">

  <div id="blog-header-text">
    See <a href="https://jarnotuovinen.com/blog">jarnotuovinen.com</a> for more blog posts outside tech. This blog contains mainly tech topics.
  </div>

  <h2 class="page-heading">Posts</h2>

  <ul class="post-list">
    {% for post in site.posts %}
      <li>
        <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>

        <h2>
          <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
        </h2>
      </li>
    {% endfor %}
  </ul>

  <p class="rss-subscribe">subscribe <a href="{{ "/feed.xml" | prepend: site.baseurl }}">via RSS</a></p>

</div>
