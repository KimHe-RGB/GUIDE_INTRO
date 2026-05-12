---
layout: default
title: GUIDE
---

{% capture readme %}{% include_relative README.md %}{% endcapture %}
{{ readme | markdownify }}
