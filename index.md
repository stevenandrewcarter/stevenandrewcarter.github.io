Here is some test stuff for my site

{% for post in site.posts %}
  <p>
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <small>{{ post.date | date_to_long_string }}</small> 
    <p>{{ post.excerpt }}</p>
  </p>
{% endfor %}
