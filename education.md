---
layout: default
---

<main>
	<section>
		<div class="section-header">
			<h2>Education</h2>
		</div>
		<ul class="section-entries">
		{% for education in site.data.education limit:5 %}
			<li>
				{{ education.qualification }}
				<span class="section-entry__subtext">
					{{ education.matriculatesOn | date:"%b %Y" }}
					{% if education.graduatesOn != nil %}
					- {{ education.graduatesOn | date:"%b %Y" }}
                    {% else %}
                    - Present
                    {% endif %}
                </span>
				<span class="section-entry__subtext">{{ education.institution }}</span>
			</li>
		{% endfor %}
		</ul>
	</section>
</main>
