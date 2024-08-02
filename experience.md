---
layout: default
---

<main>
	<section>
		<div class="section-header">
			<h2>Experience</h2>
		</div>
		<ul class="section-entries">
		{% for experience in site.data.experiences limit:5 %}
			<li>
				{{ experience.employerName }}
				<span class="section-entry__subtext">
					{{ experience.startsOn | date:"%b %Y" }}
					{% if experience.endsOn != nil %}
					- {{ experience.endsOn | date:"%b %Y" }}
					{% endif %}
			   	</span>
				<span class="section-entry__subtext">{{ experience.role }}</span>
			</li>
		{% endfor %}
		</ul>
	</section>
</main>
