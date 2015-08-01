---
layout: default
title: 分类
permalink: /pages/categories.html
---
<div>

	{% for category in site.categories %}
		{% if category[0] != '作品' %}
		<div class="col-md-12">
	      		<div class="panel panel-primary">
	        			<div class="panel-heading center" id="{{ category[0] }}" name="{{ category[0] }}">{{ category[0] }}</div>
			              {% for post in category[1] %}
			                 <a  href='{{ post.url }}'  class="list-group-item clearfix">
				            {{post.title}}
				            <span class="badge">{{ post.date | date:"%Y年%m月%d日" }}</span>
				     </a>
			               {% endfor %}
			</div>
		</div>
	 {% endif %}
	{% endfor %}

</div>
