---
layout: default
title: 标签
permalink: /pages/tags.html
---

<div class="page-tag">
	{% for tag in site.tags %}
		<a href="#{{tag[0]}}" name="{{tag[0]}}">{{ tag[0] }}({{tag[1].size}})</a>
	{% endfor %}
</div>

<div class="clear"></div>

<div>
	{% for tag in site.tags %}
	<div class="blue_box" id = "{{tag[0]}}" name="{{tag[0]}}">
		<h3 class="blue_title" >{{tag[0]}}</h3>
		<div class="box_content">
		<div class="tags">
			{% for post in tag[1] %}
				<div class="article">
					<a href="{{ post.url }}">{{ post.title }} </a>
					&nbsp;&nbsp;
					<p class="badge">{{ post.date | date:"%Y年%m月%d日" }} </p>
				</div>
			{% endfor %}
		</div>
		</div>
	</div>
	{% endfor %}
</div>
