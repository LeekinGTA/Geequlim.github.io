---
layout: default
title: 归档
permalink: /pages/archive.html
---
<div class="orange_box">
	{% for post in site.posts  %}
	    {% capture this_year %}{{ post.date | date: "%Y" }}{% endcapture %}
	    {% capture this_month %}{{ post.date | date: "%m" }}{% endcapture %}
	    {% capture next_year %}{{ post.previous.date | date: "%Y" }}{% endcapture %}
	    {% capture next_month %}{{ post.previous.date | date: "%m" }}{% endcapture %}

	    {% if forloop.first %}
	      <legend id="{{this_year}}-{{this_month}}" class="orange_title">{{this_year}}年{{this_month}}月</legend>
	      <ul>
	    {% endif %}
			<div class="box_content">
	    <li class="archive_span"><span>{{ post.date | date: "%Y年%m月%d日" }}</span> &raquo;
	      <a href="{{ post.url }}">{{ post.title }}</a>
	    </li>
			</div>
	    {% if forloop.last %}
	      </ul>
	    {% else %}
	      {% if this_year != next_year %}
	        </ul>
	        <legend id="{{next_year}}-{{next_month}}" class="brown_title">{{next_year}}年{{next_month}}月</legend>
	        <ul>
	      {% else %}
	        {% if this_month != next_month %}
	        </ul>
	        <legend id="{{next_year}}-{{next_month}}" class="blue_title">{{next_year}}年{{next_month}}月</legend>
	        <ul>
	        {% endif %}
	      {% endif %}
	    {% endif %}
	{% endfor %}
</div>
