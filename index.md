---
layout: page
title: honger05
description: 运动达人，篮球高手，游泳健将。看过许多技术类书籍，喜欢折腾前端各种技术
---
{% include JB/setup %}

{% for post in site.posts limit:10 %}
<div class = "card">
		<div  class = "date_label">
			<div class="day_month">
      			{{ post.date | date:"%m/%d" }}
      			</div>
      			<div class="year">
      			{{ post.date | date:"%Y" }}
      			</div>
      		</div>
		{{ post.content  | | split:'<!--break-->' | first }}
	<div class = "read_more">
		<a class="fa fa-link" href="{{ BASE_PATH }}{{ post.url }}">  查看全文&hellip;</a>
	</div>

</div>

{% endfor %}

{% assign len = site.posts | size %}
{% if len > 10 %}
  {% assign archive = site.pages | where:"title","Archive" %}
  <a class="waves-effect waves-light right orange darken-4" href="/archive"
		style="display:block;margin-top:20px;">
		<i class="mdi-image-style right"></i><span lang="MORE_INFO">所有文章</span>
	</a>
  {% assign archive = nil %}
{% endif %}
{% assign len = nil %}
