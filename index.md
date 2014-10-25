---
layout: default
---
<p>
  Welcome, I am Jazzi.
</p>

<p><br /><b>My Blog:</b></p>
  <ul class="posts">
    {% for post in site.posts %}
      <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>

<p><br /><b>My Project</b></p>
<ul>
  <li><a href="https://github.com/jazzi">Github</a></li>
  <li><a href="https://jazzi.github.com/goMaster/index.html"大唐双龙传</a></li>
</ul>


