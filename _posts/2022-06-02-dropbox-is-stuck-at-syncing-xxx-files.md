---
layout: post
title: "Dropbox is stuck at syncing xxx files"
subtitle: "solved through fixing hardlinks and permissions"
date: 2022-06-02 15:46:47
header-style: text
catalog: true
author: "Yuan"
tags: [Dropbox,Mac,syncing,setting]
---
{% include linksref.html %}

>Stay focused

I have used Dropbox for more than 5 yrs now. Recently, the Dropbox desktop app is stuck at 'syncing 2 files' or 'indexing 2 files'. Furthermore, this made dropbox occupy too much CPU, memory, and network bands. I have to quit Dropbox if I want to work smoothly. 

Searching online, there are a lot of similar problems discussed. Unfortunately, there is no 'summary article' to include all verified solutions, and I have to try them one by one. This is awful!

Indeed, many issues might cause this issue(BTW, Dropbox stay where they were almost for 5yrs, while other potential competitors are catching-up. I may switch to Box.com or googleDrive next year), but Dropbox seemed never come out with a robust solution.

As there is no "running log" available, I could only find this fig showing the final 'outcome' of the syncing problem:

![Sync Issues](/img/in-post/dropbox_synIssue2.png)

I tried one solution showed [here](https://www.dropboxforum.com/t5/Dropbox-files-folders/My-desktop-app-is-stuck-at-syncing-and-I-need-help-with-the/m-p/405492)
{{tip}}
trying the steps below to see if that resolves the "Syncing" status for you:
<br/><br/>
<ul>
<li>Click the Dropbox icon in your system tray/menu bar.</li>
<li>Click the profile pic/initials icon in the top right of the window.</li>
<li>Choose "Preferences..."</li>
<li>Go to the "Account" tab.</li>
<li>While holding the Space bar key (Windows/Linux) or Option/Alt key (Mac), click "Fix Hardlinks".</li>
<li>If this doesn't resolve the issue, please follow the steps again and instead click "Fix permissions". </li>
</ul>
{{end}}

After following this, Dropbox seemed to reindex all my files. I'm stilling waiting for the results.

Another issue sync issue is related to the filenames, as shown below:

![Sync Issues](/img/in-post/dropbox_synIssue1.png)

It seems that there are 2 files I could not upload to the cloud due to the filename issue.

Unfortunately, clicking 'Go to file' link won't lead you anywhere, and you can **NOT** get any information about **WHERE ARE THOSE 2 FILES**. I really hate this kind of 'let you guess' without logs or detailed hints, such an awful design!

I also tried to solve this by setting full disk access permissions to Dropbox:

1. On your computer, click the Apple menu (Apple icon) in the top left corner of your screen.
2. Click System Preferences. 
3. Click Security & Privacy.  
4. Click the Privacy tab. 
5. In the bottom left corner, click the lock icon next to Click the lock to make changes. 
Enter your Mac username and password. 
6. In the menu on the left, scroll down and click Full Disk Access. 
7. Check the box next to Dropbox. 
8. Click Later.
9. In the menu on the left, click Accessibility. 
10. Check the box next to Dropbox. 
11. Click the lock icon again. 
12. Quit and relaunch the Dropbox desktop app.

---
