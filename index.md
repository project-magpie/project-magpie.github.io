---
layout: page
title: project magpie 
tagline: Empowering digital Freedom
---
{% include magpie/setup %}

The [project magpie](https://github.com/project-magpie) can be found on github.  

    
## Blog 
<ul class="posts">
  {% for post in site.posts %}
    <li>
        <span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>
        <p>{{ post.excerpt }}</p>
	<a href="{{ BASE_PATH }}{{ post.url }}">[...]</a>  
    </li>
{% endfor %}
</ul>

