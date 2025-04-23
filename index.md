My personal website, with lots of ramblings in a sort-of-blog format. There's probably code involved, maybe one or two hot-takes about Minecraft.

If you're wondering why the page is so minimalist: I am too lazy to create web graphics or deal with HTML or CSS for more than 6 minutes at a time.

{% include micro-blog.html %}

<ul style="padding-left: 0px text-align: center;">
{% assign collections = site.collections | where_exp: "item", "item.label != 'posts'" %}
{% for collection in collections %}
  {% assign firstPost = site[collection.label] | sort: 'ordering' | first %}
  <li style="display: inline-block; padding: 10px 20px;"><a href="{{ firstPost.url | relative_url }}">{{ collection.title }}</a></li>
{% endfor %}
</ul>

---

<ul>
  {% for post in site.posts %}
    <li>
      <h2><a href="{{ post.url | relative_url }}">{{ post.date | date: "%-d %B %Y" }} - {{ post.title }}</a></h2>
      {% if post.description %}
        {{ post.description }}
      {% else %}
        {{ post.excerpt }}
      {% endif %}
    </li>
  {% endfor %}
</ul>
