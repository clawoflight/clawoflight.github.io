---
title: Bennett Piater
description: "My Blog, Random Thoughts and Notes, Personal Projects and Other Stuff."
layout: post
image:
    feature: Innsbruck.jpg
---

## Uni-Gemeinsam

Uni is tough, and it's good to have friends to work with and help each other with our studies. That's why I co-founded a collaboration group as a place to meet people, share insights and get help for debugging ;)

You can find it's homepage [here](/uni).

## Blog

My Blog is probably what you are looking for. You can click on the corresponding menu item in the top right corner, or simply [here](/blog).

// Todo: add a preview of the latest post

<div id='bump'>
    <section class="archive">
      <article class="archive-wrap">
          <ol class="post-preview-home">
             <lh><h2><span class="bb">{{ page.title }}</span></h2></lh>
              {% for post in site.posts | limit:1 %}
              <li>
                <div class="deets" itemscope itemtype="http://schema.org/BlogPosting" itemprop="blogPost">
                    <h1><a href="{{ site.url }}{{ post.url }}">{{ post.title }}</a></h1>
                    <p class="date"><time datetime="{{ post.date | date_to_xmlschema }}" itemprop="datePublished">{{ post.date | date: "%B %d, %Y" }} -- in: {{ post.category }}</a></time></p>
                    <p class="">{% if post.description %}{{ post.description  | strip_html | strip_newlines | truncate: 200 }}{% else %}{{ post.content | strip_html | strip_newlines | truncate: 200 }}{% endif %}</p>
                </div>
              </li>
              {% endfor %}
          </ol>
      </article>
    </section>
</div>
