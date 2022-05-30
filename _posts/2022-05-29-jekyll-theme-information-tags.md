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
{% include linkrefs.html %}

>Fly high

Jekyll provide flexible ways to write alert information in your markdown blogs. I choose the one contributed by [tomjoht](https://github.com/tomjoht/documentation-theme-jekyll), which is the updated. Still, there is an order version, from [parpersson](https://github.com/parpersson/Manualmall), which was probably forked from tomjoht. 

## Older version
Based on the order version, in order to use these alerts or callouts, put this include at the top of your page, just below your frontmatter, check [here](http://parpersson.github.io/Manualmall/alerts/):

```r
{% include note.html %}
```

Then, you could add note using the syntax:

```r
{{note}} your note {{end}}
#Here, could change 'note' into 'tip','warning', or 'important'
```

{{note}} My note Test {{end}}

{{tip}} your tip {{end}}

{{warning}} your warning {{end}}

{{important}} your important info {{end}}

And callouts below:
```r
{{callout_default}} your **callout_default** content {{end}}

```

{{callout_default}} your **callout_default** content {{end}}

{{callout_primary}} your callout_primary content {{end}}

{{callout_success}} your callout_success content {{end}}

{{callout_primary}} your callout_primary content {{end}}

{{callout_warning}} your callout_warning content {{end}}

{{callout_info}} your callout_info content {{end}}

Alternatively, you could also include note.html where you want it:

```bash
{% include note.html content="Note alalallalala" %}
```

## Alerts

{% include note.html content="Note alalallalala" %}

{{note}} Note again {{end}}

---
