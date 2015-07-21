---
layout: default
title: 作品
permalink: /pages/projects.html
---


<div class="bookpiclist">
	{% for page in site.posts %}
			{% if page.categories[0] == '作品' %}
				{% for project in page.projects %}
				<div class="bookpic">
					<div class="bookpic-wrapper">
						<a href="{{page.url}}#{{ project.title }}"><img src="{{ project.cover }}" alt="cover">
							<div class="booklabel">
                  <div class="label-text center">
                  {% if project.status == '已完成' %}
									<span class="label label-success">{{ project.title }}</span>
								    {% elsif project.status == '开发中' %}
									<span class="label label-info">{{ project.title }}</span>
								    {% elsif project.status == '计划中' %}
									<span class="label label-default">{{ project.title }}</span>
								    {% endif %}
					        </div>
					        <div class="label-bg"></div>
					    </div>
						</a>
					</div>
				</div>
				{% endfor %}
			{% endif %}
	{% endfor %}
</div>
