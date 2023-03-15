---
layout: index
---

{% for collection in site.collections %}
    {% if collection.label != 'posts' %}
<details markdown="1" open="true" class="index_details">
<summary>{{collection.label}}</summary>

        {% for item in site[collection.label] %}
* ### [{{item.title}}]({{ item.url }})
        {% endfor %}
</details>
    {% endif %}
{% endfor %}