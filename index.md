---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
title: main()
---

<section class="posts">

<h1 style="font-family: system-ui;">
Welcome
</h1>

<ul>
{% for post in site.posts %}
<li>
<code>
{
    .date = <time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%Y-%m-%d" }}</time>,
    .title = "<a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>"s,
}
</code>

</li>
{% endfor %}
</ul>
</section>
