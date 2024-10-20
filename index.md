My personal website, with lots of ramblings in a sort-of-blog format. There's probably code involved, maybe one or two hot-takes about Minecraft.

If you're wondering why the page is so minimalist: I am too lazy to create web graphics or deal with HTML or CSS for more than 6 minutes at a time.

<div id="micro-blog"></div>
<script type="text/javascript">
  const FEED_URL = 'https://retrodev.social/api/v1/timelines/public?local=true';
  fetch(FEED_URL)
    .then(res => res.json())
    .then(data => data.find(post => post.account.username === "xilefian" && post.in_reply_to_id == null && post.in_reply_to_account_id == null))
    .then(post => {
      document.head.insertAdjacentHTML("beforeend", `
        <link rel="stylesheet" href="assets/css/retrodev.css">
        <link rel="stylesheet" href="https://retrodev.social/assets/Fork-Awesome/css/fork-awesome.min.css">
      `);
      const account = post.account;
      const createdAt = new Date(post.created_at);
      const postLanguage = new Intl.DisplayNames(undefined, { type: 'language' }).of(post.language)
      let html = `
        <article class="status expanded" id="${post.id}" role="region" aria-label="@${account.username}, ${createdAt.toLocaleDateString(undefined, {month: 'short', day: 'numeric', hour: 'numeric', minute: 'numeric', hourCycle: 'h23'})}, language ${postLanguage}, ${post.replies_count} reply, ${post.favourites_count} favourite">
          <header class="status-header">
              <address>
                  <a href="${account.url}" rel="author" title="Open profile">
                      <picture class="avatar" aria-hidden="true" style="visibility:hidden">
                          <source srcset="${account.avatar_static}" type="image/jpeg" media="(prefers-reduced-motion: reduce)">
                          <img src="${account.avatar}" alt="Avatar for ${account.username}" title="Avatar for ${account.username}">
                      </picture>
                      <div class="author-strap">
                          <span class="displayname text-cutoff">${account.display_name}</span>
                          <span class="sr-only">,</span>
                          <span class="username text-cutoff">@${account.username}</span>
                      </div>
                      <span class="sr-only">(open profile)</span>
                  </a>
              </address>
          </header>
          <div class="status-body">
              <div class="text">        
                  <div class="content" lang="${post.language}">
                      ${post.content}
                  </div>
              </div>
          </div>
          <aside class="status-info" aria-hidden="true">    
              <dl class="status-stats">
                  <div class="stats-grouping">
                      <div class="stats-item published-at text-cutoff">
                          <dt class="sr-only">Published</dt>
                          <dd>
                              <time datetime="${post.created_at}">${createdAt.toLocaleDateString(undefined, {year: 'numeric', month: 'short', day: 'numeric', hour: 'numeric', minute: 'numeric', hourCycle: 'h23'})}</time>
                          </dd>
                      </div>
                      <div class="stats-grouping">
                          <div class="stats-item" title="Replies">
                              <dt>
                                  <span class="sr-only">Replies</span>
                                  <i class="fa fa-reply-all" aria-hidden="true"></i>
                              </dt>
                              <dd>${post.replies_count}</dd>
                          </div>
                          <div class="stats-item" title="Faves">
                              <dt>
                                  <span class="sr-only">Favourites</span>
                                  <i class="fa fa-star" aria-hidden="true"></i>
                              </dt>
                              <dd>${post.favourites_count}</dd>
                          </div>
                          <div class="stats-item" title="Boosts">
                              <dt>
                                  <span class="sr-only">Reblogs</span>
                                  <i class="fa fa-retweet" aria-hidden="true"></i>
                              </dt>
                              <dd>${post.reblogs_count}</dd>
                          </div>
                      </div>
                  </div>
                  <div class="stats-item language" title="${postLanguage}">
                      <dt class="sr-only">Language</dt>
                      <dd>
                          <span class="sr-only">${postLanguage}</span>
                          <span aria-hidden="true">${post.language}</span>
                      </dd>
                  </div>
              </dl>
          </aside>
          <a href="${post.url}" class="status-link" data-nosnippet="" title="Open thread at this post">
              Open thread at this post
          </a>
        </article>
      `;
      document.getElementById('micro-blog').innerHTML = html;
    });
</script>

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
