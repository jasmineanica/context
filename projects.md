---
layout: page
title: Projects
permalink: /projects/
---

Small projects where I take a data structure or algorithm I've been studying and build something real with it. Each one links to the source on GitHub and (where applicable) a live demo.

<style>
  .project-list { list-style: none; padding: 0; margin: 0; }
  .project-list li {
    border: 1px solid #cdd8c1;
    border-radius: 6px;
    padding: 1rem 1.25rem;
    margin-bottom: 1rem;
    background: #fafcf6;
  }
  .project-list h3 { margin: 0 0 0.25rem 0; }
  .project-list .tagline { color: #4a553f; margin: 0 0 0.75rem 0; }
  .project-list .concepts { margin: 0 0 0.75rem 0; }
  .project-list .concept-tag {
    display: inline-block;
    font-size: 0.8rem;
    padding: 0.15rem 0.55rem;
    margin: 0 0.35rem 0.3rem 0;
    background: #e6ecd9;
    color: #3d5430;
    border-radius: 999px;
  }
  .project-list .links a {
    margin-right: 0.75rem;
    font-weight: 600;
  }
  .project-list .status-wip { color: #8a6a1f; font-size: 0.85rem; }
  .project-list .status-archived { color: #777; font-size: 0.85rem; }
</style>

<ul class="project-list">
{% for project in site.data.projects %}
  <li>
    <h3>{{ project.name }}{% if project.status == "wip" %} <span class="status-wip">(in progress)</span>{% elsif project.status == "archived" %} <span class="status-archived">(archived)</span>{% endif %}</h3>
    <p class="tagline">{{ project.tagline }}</p>
    <p class="concepts">
      {% for concept in project.concepts %}
        <span class="concept-tag">{{ concept }}</span>
      {% endfor %}
    </p>
    <p class="links">
      {% if project.repo %}<a href="{{ project.repo }}">Source</a>{% endif %}
      {% if project.demo and project.status == "live" %}<a href="{{ project.demo }}">Live demo</a>{% endif %}
    </p>
  </li>
{% endfor %}
</ul>

---

Want to know how I scaffold these? See [adding a project]({{ "/docs/adding-a-project/" | relative_url }}).
