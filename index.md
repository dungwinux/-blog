---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

<section class="posts">

<h1 style="font-family: system-ui;">
Welcome
</h1>

<ul class="address-table">
{% for post in site.posts %}
<li class="address-entry">
    <i class="address-entry-time">
        <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%Y`%m%d" }}</time>
    </i>
    <div class="address-entry-text">
        <b><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></b>
        <div style="font-size:small">{{ post.excerpt }}</div>
    </div>
</li>
{% endfor %}
</ul>
</section>
