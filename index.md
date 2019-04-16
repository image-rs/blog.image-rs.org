# The image-rs organization

Open-source, image related Rust crates since 2019¹.

¹: split from Piston in 2019, many crates are older.

<div class="col-md-12 image-rs-posts">
  {% for post in site.posts %}
    <div class="image-rs-post">
      <h1 class="post-title">
        <a href="{{ post.url }}">
          {{ post.title }}
        </a>
      </h1>
      <h2 class="post-info">
        <span class="post-info-date">
          <i class="fa fa-clock-o"></i>
          {{ post.date | date: "%B %-d, %Y" }}
        </span>
        <span class="post-info-author">
          <i class="fa fa-user"></i>
          <a href="https://www.github.com/{{ post.author }}" target="_blank">{{ post.author }}</a>
        </span>
      </h2>

      <div class="post-summary">
        {{ post.excerpt }}
      </div>
    </div>
  {% endfor %}
</div>
