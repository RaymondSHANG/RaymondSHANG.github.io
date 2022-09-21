---
layout: post
title: "git multiprofile settings"
subtitle: "multithread roles"
date: 2022-09-21 11:09:03
header-style: text
catalog: true
author: "Yuan"
tags: [git,profile]
---
{% include linksref.html %}
> Time management

# Default git config file
Git profile file is at <code> ~/.gitconfig </code>, which has a content like this:

```git
[credential]
    helper = cache
[user]
	name = Yuan Shang
	email = shangyuan5000@gmail.com
```

#

When you have multiple roles collaborating with different entities, you may consider create multiple git profile.
---
