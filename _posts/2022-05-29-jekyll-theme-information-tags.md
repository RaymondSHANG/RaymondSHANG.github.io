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
{% include linksref.html %}

>Fly high

Jekyll provide flexible ways to write alert information in your markdown blogs. I choose the one contributed by [tomjoht](https://github.com/tomjoht/documentation-theme-jekyll), which is the updated. Still, there is an order version, from [parpersson](https://github.com/parpersson/Manualmall), which was probably forked from tomjoht. 

## Older version
Based on the order version, in order to use these alerts or callouts, put this include at the top of your page, just below your frontmatter, check [here](http://parpersson.github.io/Manualmall/alerts/):

```
{%raw%}{% include linksref.html %}{% endraw%}
```

Then, you could add note using the syntax:

```
{%raw%}{{note}} your note {{end}}{% endraw%}
#Here, could change 'note' into 'tip','warning', or 'important'
```

{{note}} My note Test using capture {{end}}

{{tip}} your tip {{end}}

{{warning}} your warning {{end}}

{{important}} your important info {{end}}

And callouts below:
```
{%raw%}{{callout_default}} your **callout_default** content {{end}}{% endraw%}

```

{{callout_default}} your **callout_default** content {{end}}

{{callout_primary}} your callout_primary content {{end}}

{{callout_success}} your callout_success content {{end}}

{{callout_primary}} your callout_primary content {{end}}

{{callout_warning}} your callout_warning content {{end}}

{{callout_info}} your callout_info content {{end}}




## Newversion
Alternatively, you could also include note.html where you want it. The new version is based from [here](https://idratherbewriting.com/documentation-theme-jekyll/mydoc_alerts.html#about-alerts):

```
{%raw%}{% include note.html content="Note alalallalala" %}{% endraw%}
```

{% include note.html content="Note alalallalala" %}

And note with multiple lines using `<br/><br/>` tags:

{% include note.html content="This is my note. All the content I type here is treated as a single paragraph. <br/><br/> Now I'm typing on a  new line." %}


Or if you want to include codes: 

{{site.data.alerts.note}}
<p>This is my note.</p>
<pre>
def foo(x):<br>
&nbsp;&nbsp;&nbsp;&nbsp;return x+1
</pre>
{{site.data.alerts.end}}

code for above:

```
{%raw%}
{{site.data.alerts.note}}
<p>This is my note.</p>
<pre>
def foo(x):<br>
&nbsp;&nbsp;&nbsp;&nbsp;return x+1
</pre>
{{site.data.alerts.end}}
{% endraw%}
```
Other alerts:

{% include tip.html content="tip alalallalala" %}

{% include warning.html content="warnings alalallalala" %}

{% include important.html content="important alalallalala" %}

And callouts:

{% include callout.html content="This is my callout. It has a border on the left whose color you define by passing a type parameter. I typically use this style of callout when I have more information that I want to share, often spanning multiple paragraphs. " type="primary" %} 

Callouts with multiple lines:

{% include callout.html content="**Important information**: This is my callout. It has a border on the left whose color you define by passing a type parameter. I typically use this style of callout when I have more information that I want to share, often spanning multiple paragraphs. <br/><br/>Here I am starting a new paragraph, because I have lots of information to share. You may wonder why I'm using line breaks instead of paragraph tags. This is because Kramdown processes the Markdown here as a span rather than a div (for whatever reason). Be grateful that you can be using Markdown at all inside of HTML. That's usually not allowed in Markdown syntax, but it's allowed here." type="primary" %} 

```
{%raw%}{% include callout.html content="**Important information**: This is my callout. <br/><br/>Here I am starting a new paragraph." type="primary" %}{% endraw%}
# type could be danger, default, primary, success, info, and warning.
```

Callouts summary:

| Property | description                                                                                              |
| -------- | -------------------------------------------------------------------------------------------------------- |
| content  | The content for the callout.                                                                             |
| type     | The style for the callout. Options are `danger`, `default`, `primary`, `success`, `info`, and `warning`. |


{% include callout.html content="This is danger callout. " type="danger" %} 

{% include callout.html content="This is default callout. " type="default" %}

{% include callout.html content="This is primary callout. " type="primary" %}

{% include callout.html content="This is success callout. " type="success" %}

{% include callout.html content="This is info callout. " type="info" %}

{% include callout.html content="This is warning callout. " type="warning" %}


---
