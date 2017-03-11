Here is some test stuff for my site

{% for post in site.posts %}
  <p>
    <a href="{{ post.url }}">{{post.date}} {{ post.title }}</a>
    <pre>{{ post.excerpt }}</pre>
  </p>
{% endfor %}
