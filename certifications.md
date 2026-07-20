---
layout: default
---
{% assign sortedCertifications =
   site.data.certifications
   | sort: "name"
   | sort: "expiresOn"
   | sort: "startsOn"
   | reverse -%}

<article>
	<section>
		<div class="section-header">
			<h2>Certifications</h2>
		</div>
		<ul class="section-entries">
		{% for certification in sortedCertifications -%}
			<li>
				{{ certification.name }}
				<span class="section-entry__subtext">
                    {% assign certificationDateRange = certification.startsOn | date:"%b %Y" -%}
					{% if certification.expiresOn != nil -%}
                      {% assign certificationExpiresOnFormatted = certification.expiresOn | date:"%b %Y" -%}
                      {% assign certificationDateRange =
                         certificationDateRange
                         | append: " - "
                         | append: certificationExpiresOnFormatted -%}
				    {% endif -%}
                    {{ certificationDateRange }}
				</span>
			</li>
		{% endfor -%}
		</ul>
	</section>
</article>
