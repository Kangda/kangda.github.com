---
layout: home
---

<div class="index-content blog">
	<div class="navigator">
		<ul class="artical-cate">
            <li class="on"><a href="/"><span>Blog</span></a></li>
            <li style="text-align:right"><a href="/about"><span>About</span></a></li>
        </ul>

        <div class="cate-bar"><span id="cateBar"></span></div>
	</div>
    <div class="section">
		<article class="content">
			<section class="post">
				<ul class="listing">
				{% for post in site.posts %}
					{% capture y %}{{post.date | date:"%Y"}}{% endcapture %}
					{% if year != y %}
						{% assign year = y %}
						<li class="listing-seperator">{{ y }}</li>
					{% endif %}
					<li class="listing-item">
						<time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time>
						<a href="{{ site.url }}{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
					</li>
				{% endfor %}
				</ul>
			</section>
		</article>
		
		<!--
        <ul class="artical-list">
        {% for post in site.categories.blog %}
            <li>
				<h3>{{ post.date }}</h3>
                <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
                <div class="title-desc">{{ post.description }}</div>
            </li>
        {% endfor %}
        </ul>
		-->
    </div>
    <div class="aside">
    </div>
</div>
