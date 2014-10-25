---
layout: default
---
<p>
  I'm jazzi, here is all about my personal stuff and thinking of the world, if something here affend you, please move on, thanks.
</p>

<p><br /><b>My Blog:</b></p>
  <ul class="posts">
    {% for post in site.posts %}
      <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>