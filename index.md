My personal website, with lots of ramblings in a sort-of-blog format. There's probably code involved, maybe one or two hot-takes about Minecraft.

If you're wondering why the page is so minimalist: I am too lazy to create web graphics or deal with HTML or CSS for more than 6 minutes at a time.

<ol>
{% assign collections = site.collections | where_exp: "item", "item.label != 'posts'" %}
{% for collection in collections %}
  <li>
    <a>{{ collection.title }}</a>
    <ol>
    {% assign posts = site[collection.label] | sort: 'ordering' %}
    {% for post in posts %}
      <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
    {% endfor %}
    </ol>
  </li>
{% endfor %}
</ol>

<ul>
  {% for post in site.posts %}
    <li>
      <h2><a href="{{ post.url | relative_url }}">{{ post.date | date: "%-d %B %Y" }} - {{ post.title }}</a></h2>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
