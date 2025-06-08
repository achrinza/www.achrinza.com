---
layout: default
---

<div>
<article itemprop="mainContentOfPage">
  <section>
    <div class="h-card">
      <h1 class="p-name name">Rifa Achrinza</h1>
	  <h2 class="p-role roles">Software engineer, aspiring infosec engineer</h2>
	  <p class="short-intro">I am a <i>software engineer, full-stack web developer, open source contributor, aspiring infosec engineer, analytical problem solver, forever student, and pragmatic idealist</i>.</p>
	  
	  <p>I enjoy tinkering and experimenting with new and old. I am motivated by challenging projects with self-guided research, and learning how things work under the hood.</p>
	  
	  <p>This is my personal space, where I share projects, guides and research.</p>
    </div>
    </section>
	
	<section>
		<div class="section-header">
			<h2>Blog posts</h2>
			<a class="section-header__view-all" href="/blog">View all posts</a>
		</div>
		<ul class="section-entries">
			{% for post in site.posts limit:5 %}
			<li>
				<a href="{{ post.url }}">
					{{ post.title }}
					<span class="section-entry__subtext">{{ post.date | date:"%b %Y" }}</span>
				</a>
			</li>
			{% endfor %}
		</ul>
	</section>
	
	<section>
		<div class="section-header">
			<h2>Experience</h2>
			<a class="section-header__view-all" href="/experience">View all experience</a>
		</div>
	<ul class="section-entries">
		{% for experience in site.data.experiences limit:5 %}
		<li>
		    {{ experience.employerName }}
			<span class="section-entry__subtext">
				{{ experience.startsOn | date:"%b %Y" }}
				{% if experience.endsOn != nil %}
				- {{ experience.endsOn | date:"%b %Y" }}
                {% else %}
                - Present
				{% endif %}
				</span>
			<span class="section-entry__subtext">{{ experience.role }}</span>
		</li>
		{% endfor %}
	</ul>
	</section>
	<section>
		<div class="section-header">
			<h2>Certifications</h2>
			<a class="section-header__view-all" href="/certifications">View all certifications</a>
		</div>
	<ul class="section-entries">
		{% assign featured_certifications = site.data.certifications | where:"featured", true %}
		{% for certification in featured_certifications %}
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
	<section>
		<div class="section-header">
			<h2>Education</h2>
			<a class="section-header__view-all" href="/education">View all acadmic qualifications</a>
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
	
	<section>
        <h2>Get in touch</h2>
		<p>If you an to connect with me or just say  hi, reach out on social
        media or preferrably send email.</p>
        <p>rifa@[my surname].com</p>
        <ul class="social-links">
			<li><a class="btn" href="https://linkedin.com/in/achrinza"><i class="fa-brands fa-linkedin"></i></a></li>
			<li><a class="btn" href="https://github.com/achrinza"><i class="fa-brands fa-github"></i></a></li>
			<li><a class="btn" href="https://instagram.com/achrinza"><i class="fa-brands fa-instagram"></i></a></li>
			<li><a class="btn" href="https://twitter.com/achrinza"><i class="fa-brands fa-twitter"></i></a></li>
		</ul>
    </section>
  </article>
</div>
