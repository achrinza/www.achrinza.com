---
layout: default
---
<article>
	<section>
		<div class="section-header">
			<h2>Blog posts</h2>
		</div>
		<ul class="section-entries">
		{% for post in site.posts %}
			<li>
				<a href="{{ post.url }}">
					{{ post.title }}
					<span class="section-entry__subtext">{{ post.date | date:"%b %Y" }}</span>
				</a>
			</li>
		{% endfor %}
		</ul>
	</section>
</article>
