---
title: "Projects"
permalink: /projects/
layout: single
classes: wide
sort_by: order
---

{% assign entries = site.projects %}

{% if page.sort_by %}
  {% assign entries = entries | sort: page.sort_by %}
{% endif %}

<h2>Recent projects</h2>

<div class="entries-grid">
  {% for post in entries limit: 3 %}
    {% if post.header.teaser %}
      {% capture teaser %}{{ post.header.teaser }}{% endcapture %}
    {% else %}
      {% assign teaser = site.teaser %}
    {% endif %}
    {% assign title = post.title %}
    <div class="grid__item">
      <article class="archive__item" itemscope itemtype="https://schema.org/CreativeWork"{% if post.locale %} lang="{{ post.locale }}"{% endif %}>
        {% if teaser %}
          <div class="archive__item-teaser">
            <img src="{{ teaser | relative_url }}" alt="">
          </div>
        {% endif %}
        <h2 class="archive__item-title no_toc" itemprop="headline">
          {% if post.link %}
            <a href="{{ post.link }}">{{ title }}</a> <a href="{{ post.url | relative_url }}" rel="permalink"><i class="fas fa-link" aria-hidden="true" title="permalink"></i><span class="sr-only">Permalink</span></a>
          {% else %}
            <a href="{{ post.url | relative_url }}" rel="permalink">{{ title }}</a>
          {% endif %}
        </h2>
          <p class="page__meta">
            {% if post.show_date and post.year %}
              {% assign date = post.year %}
              <span class="page__meta-date">
                <i class="far {% if include.type == 'grid' and post.read_time and post.show_date %}fa-fw {% endif %}fa-calendar-alt" aria-hidden="true"></i>
                <time datetime="{{ date | date_to_xmlschema }}">{{ date }}</time>
              </span>
            {% endif %}
          </p>
        {% if post.excerpt %}<p class="archive__item-excerpt" itemprop="description">{{ post.excerpt | markdownify | strip_html | truncate: 160 }}</p>{% endif %}
      </article>
    </div>
  {% endfor %}
</div>

<h2>High school projects</h2>

<div class="entries-grid">
  {% for post in entries offset: 3 %}
    {% if post.header.teaser %}
      {% capture teaser %}{{ post.header.teaser }}{% endcapture %}
    {% else %}
      {% assign teaser = site.teaser %}
    {% endif %}
    {% assign title = post.title %}
    <div class="grid__item">
      <article class="archive__item" itemscope itemtype="https://schema.org/CreativeWork"{% if post.locale %} lang="{{ post.locale }}"{% endif %}>
        {% if teaser %}
          <div class="archive__item-teaser">
            <img src="{{ teaser | relative_url }}" alt="">
          </div>
        {% endif %}
        <h2 class="archive__item-title no_toc" itemprop="headline">
          {% if post.link %}
            <a href="{{ post.link }}">{{ title }}</a> <a href="{{ post.url | relative_url }}" rel="permalink"><i class="fas fa-link" aria-hidden="true" title="permalink"></i><span class="sr-only">Permalink</span></a>
          {% else %}
            <a href="{{ post.url | relative_url }}" rel="permalink">{{ title }}</a>
          {% endif %}
        </h2>
          <p class="page__meta">
            {% if post.show_date and post.year %}
              {% assign date = post.year %}
              <span class="page__meta-date">
                <i class="far {% if include.type == 'grid' and post.read_time and post.show_date %}fa-fw {% endif %}fa-calendar-alt" aria-hidden="true"></i>
                <time datetime="{{ date | date_to_xmlschema }}">{{ date }}</time>
              </span>
            {% endif %}
          </p>
        {% if post.excerpt %}<p class="archive__item-excerpt" itemprop="description">{{ post.excerpt | markdownify | strip_html | truncate: 160 }}</p>{% endif %}
      </article>
    </div>
  {% endfor %}
</div>

