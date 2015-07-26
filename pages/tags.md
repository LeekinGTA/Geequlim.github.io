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
		<div class="tags">
			{% for post in tag[1] %}
				<div class="article">
					<span class="datetime">{{ post.date | date:"%Y-%m-%d" }} </span>&raquo;
					<a href="{{ post.url }}">{{ post.title }}</a>
				</div>
			{% endfor %}
		</div>
	</div>
	{% endfor %}
</div>
