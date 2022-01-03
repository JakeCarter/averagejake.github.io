<section class="posts">
	{% for post in paginator.posts %}
	<article class="post">
		<h2 class="title">
			<a href="{{ post.url }}">{{ post.title }}</a>
		</h2>
		<div class="date">{{ post.date | date: site.date_format }}</div>
		<div class="summary">
			{% if post.vimeo %}
			<div class="video-container">
				<iframe src="https://player.vimeo.com/video/{{ post.vimeo }}" frameborder="0" width="500" height="281" webkitAllowFullScreen mozallowfullscreen allowFullScreen></iframe>
			</div>
			{% else %}
				{{ post.excerpt }}
			{% endif %}
		</div>
		<a class="continue-reading-link" href="{{ post.url | absolute_url }}">more…</a>
	</article>
	{% endfor %}
</section>
