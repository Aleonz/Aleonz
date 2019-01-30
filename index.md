---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

<div class="home">
  <div class="posts card">
        <div class="card-content white-text">

            <ul class="post-list">
                {% for post in site.posts %}
                <li>
                    <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>     
                    <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>       
                </li>
                {% endfor %}
            </ul>

        </div>
   </div>

</div>