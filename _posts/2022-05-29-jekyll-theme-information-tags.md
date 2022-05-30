---
layout: post
title: "jekyll theme information tags"
subtitle: "note warning tip important callout"
date: 2022-05-29 19:08:48
header-style: text
catalog: true
author: "Yuan"
tags: [jekyll,markdown,note,warning,tip,important,callout]
---
{% include note.html %}

>Fly high

In order to use these alerts or callouts, put this include at the top of your page, just below your frontmatter:

```
{% include note.html %}
```

Then, you could add note using the syntax:

```
{{note}} your note {{end}}
```



Alternatively, you could also include note.html where you want it:

```
{% include note.html content="Note alalallalala" %}
```
## Alerts

{% include note.html content="Note alalallalala" %}

{{note}} Note again {{end}}

---
