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

# Multiple User Profile Setting
When you have multiple roles collaborating with different entities, you may consider create multiple git profile.

```git
[credential]
    helper = cache

# Default name and email
[user]
	name = Yuan Shang
	email = shangyuan5000@gmail.com
# Other Profiles
[includeIf "gitdir:~/Dropbox/Personal/project1/"]
    path = ~/Dropbox/Personal/project1/git-project1.conf
[includeIf "gitdir:~/Dropbox/ABClab/project2/"]
    path = ~/Dropbox/ABClab/project2/git-project2.conf
```

In <code>~/Dropbox/Personal/project1/git-project1.conf</code>, I have:
```git
[user]
    name = yshang
    email = yshang@xxx.com
```

---
