---
layout: post
title: "jekyll markdown tabs"
subtitle: "Using Navtabs"
date: 2022-05-29 12:33:26
header-style: text
catalog: true
author: "Yuan"
tags: [github,markdown,Navtabs]
---
>The palest ink is better than the best memory

#Github markdown tabs

## Navtabs demo

The following is a [demo](https://raw.githubusercontent.com/tomjoht/documentation-theme-jekyll/gh-pages/pages/mydoc/mydoc_navtabs.md) of a navtab. 

<ul id="profileTabs" class="nav nav-tabs">
    <li class="active"><a class="noCrossRef" href="#profile" data-toggle="tab">Profile</a></li>
    <li><a class="noCrossRef" href="#about" data-toggle="tab">About</a></li>
    <li><a class="noCrossRef" href="#bash" data-toggle="tab">Using bash</a></li>
    <li><a class="noCrossRef" href="#python" data-toggle="tab">Using python</a></li>
</ul>

<div class="tab-content">
<div role="tabpanel" class="tab-pane active" id="profile" markdown="1">
## Profile

Praesent sit amet fermentum leo. Aliquam feugiat, 

1.  nibh in u ltrices mattis
2.  felis ipsum venenatis metus, vel vehicula libero mauris a enim. Sed placerat est ac lectus vestibulum tempor. 
    * Quisque ut condimentum massa. 
    * ut condimentum massa. 

> Proin venenatis leo id urna cursus blandit. Vivamus sit amet hendrerit metus.
</div>

<div role="tabpanel" class="tab-pane" id="about">
    <h2>About</h2>
    <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Aliquam vel sollicitudin felis. Sed eu arcu sed ipsum semper luctus eu a tortor. Suspendisse id leo eu metus laoreet varius. Mauris consequat accumsan ex, a iaculis metus fermentum a. Praesent sit amet fermentum leo. Aliquam feugiat, nibh in u ltrices mattis, felis ipsum venenatis metus, vel vehicula libero mauris a enim. Sed placerat est ac lectus vestibulum tempor. Quisque ut condimentum massa. Proin venenatis leo id urna cursus blandit. Vivamus sit amet hendrerit metus.about</p>
</div>

<div role="tabpanel" class="tab-pane" id="bash" markdown="1">

```bash
echo "Hello world"
```
</div>

<div role="tabpanel" class="tab-pane" id="python" markdown="1">

```python
name="world"
print(f"hello,{name}")
```
</div>

</div>
---
