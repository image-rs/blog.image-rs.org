---
---

<div class="col-md-12 image-rs-posts">
	{% for post in site.posts %}
	<article class="image-rs-post">
		<h1 class="post-title">
			<a href="{{ post.url }}" class="image-rs-internal-link">{{ post.title }}</a>
		</h1>

		<section class="post-info" role="note">
			{% include post-info.html author=post.author date=post.date %}
		</section>

		<section class="post-content">
			{{ post.excerpt }}
		</section>

		<p>
			<a href="{{ post.url }}" class="image-rs-internal-link">Continue reading &#8230;</a>
		</p>
	</article>
	{% endfor %}
</div>
