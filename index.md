Here is some test stuff for my site

{% for post in site.posts %}
  <p>
    <h3><a href="{{ post.url }}">{{post.date}} {{ post.title }}</a></h3>
    <p>{{ post.excerpt }}</p>
  </p>
{% endfor %}
