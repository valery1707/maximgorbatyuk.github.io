---
layout: page
title: Tags
---

<div class="">
    {%- capture site_tags -%}
    {%- for tag in site.tags -%}
        {{- tag | first -}}{%- unless forloop.last -%},{%- endunless -%}
    {%- endfor -%}
    {%- endcapture -%}
    {%- assign tags_list = site_tags | split:',' | sort -%}

    {%- for tag in tags_list -%}
        <div class="d-inline-block">
            <a href="#{{- tag -}}" class="mr-1 tag-btn">#{{- tag -}}&nbsp;({{site.tags[tag].size}})</a>
        </div>
    {%- endfor -%}
</div>


<div id="full-tags-list">
{%- for tag in tags_list -%}
    <h2 id="{{- tag -}}" class="linked-section">
        #{{- tag -}}&nbsp;({{site.tags[tag].size}})
    </h2>
    <ul class="posts">
        {%- for post in site.tags[tag] -%}

            <li itemscope>
              <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
              <p class="post-date"><span><i class="fa fa-calendar" aria-hidden="true"></i> {{ post.date | date: "%B %-d" }} - <i class="fa fa-clock-o" aria-hidden="true"></i> {% include read-time.html %}</span></p>
            </li>

        {%- endfor -%}
    </ul>
{%- endfor -%}
</div>
