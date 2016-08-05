---
layout: default
---

<body>
  <div class="index-wrapper">
    <div class="aside">
      <div class="info-card">
        <h1>Young</h1>
        <a href="http://weibo.com/yufeiyang1995" target="_blank"><img src="http://www.weibo.com/favicon.ico" alt="" width="25"/></a>
        <a href="https://github.com/yufeiyang1995" target="_blank"><img src="http://github.com/favicon.ico" alt="" width="22"/></a>
        <a href="https://www.zhihu.com/people/yang-yu-fei-61-46" target="_blank"><img src="http://www.zhihu.com/favicon.ico" alt="" width="22"/></a>
      </div>
      <div id="particles-js"></div>
    </div>

    <div class="index-content">
      <ul class="artical-list">
        {% for post in site.categories.blog %}
        <li>
          <a href="{{ post.url }}" class="title">{{ post.title }}</a>
          <div class="title-desc">{{ post.description }}</div>
        </li>
        {% endfor %}
      </ul>
    </div>
  </div>
</body>
