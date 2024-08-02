---
layout: default
---
<article>
	<section>
		<div class="section-header">
			<h2>Certifications</h2>
		</div>
		<ul class="section-entries">
		{% for certification in site.data.certifications %}
			<li>
				{{ certification.name }}
				<span class="section-entry__subtext">
					{{ certification.startsOn | date:"%b %Y" }}
					{% if certification.expiresOn != nil %}
				    - {{ certification.expiresOn | date:"%b %Y" }}
					{% endif %}
				</span>
			</li>
		{% endfor %}
		</ul>
	</section>
</article>
