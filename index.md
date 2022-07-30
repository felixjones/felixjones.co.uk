My personal website, with lots of ramblings in a sort of blog format. There's probably code involved, maybe some hot-takes.

If this website looks minimalist, well it's because it is. Not because I believe in minimalism, but because these days I am too lazy to create web graphics or deal with HTML or CSS for more than 6 minutes at a time.

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
