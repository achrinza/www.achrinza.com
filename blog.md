---
layout: default
titleOverride: Rifa's Blog
htmlClassAppend:
- h-feed
- hfeed
---
<article>
	<section>
		<div class="section-header">
			<h2>Blog posts</h2>
		</div>
        <div class="section-subheader">
          <a class="section-subheader__item"
             data-umami-event="feed-subscribe"
             data-umami-event-feed-type="rss"
             href="{{"/blog/rss.xml" | absolute_url }}">RSS</a>
          <a class="section-subheader__item"
             data-umami-event="feed-subscribe"
             data-umami-event-feed-type="atom"
             href="{{"/blog/atom.xml" | absolute_url }}">Atom</a>
          <a class="section-subheader__item"
             data-umami-event="feed-subscribe"
             data-umami-event-feed-type="jsonfeed"
             href="{{"/blog/feed.json" | absolute_url }}">JSON Feed</a>
        </div>
        
		<ul class="section-entries">
		{% for post in site.posts %}
		<li class="h-entry hentry">
          <article>
            <a class="u-url"
               rel="bookmark"
               href="{{ post.url }}">
              <span class="p-name entry-title">{{ post.title }}</span>
              <time class="dt-published published section-entry__subtext"
                    datetime="{{ post.date }}">{{ post.date | date:"%b %Y" }}</time>
            </a>
          </article>
        </li>
		{% endfor %}
		</ul>
	</section>
</article>
