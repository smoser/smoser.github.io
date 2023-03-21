## blog entries
<ul>
  {% for post in site.page %}
    <li>
      <a href="{{ page.permalink }}">{{ page.title }}</a>
    </li>
  {% endfor %}
</ul>
