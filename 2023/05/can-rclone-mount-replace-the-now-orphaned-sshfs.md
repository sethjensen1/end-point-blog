---
title: "Can rclone mount replace the now orphaned sshfs?"
author: Seth Jensen
date: 2023-05-26
---

When looking into SSHFS after one of my coworkers wrote [a blog post about it](), I was surprised to find that SSHFS was orphaned in 2022. Software versions in package managers are often more than a year out of date, so for a while, using its last release won't be much different than installing any other app through `apt` or `yum`.

But if no one picks up development on an ongoing basis, using SSHFS will become less of a viable option.

One replacement I saw on ArchWiki was using rclone's `mount` command, which behaves in a similar way to SSHFS. I tried mounting several servers I regularly use to see how rclone mount compares to SSHFS, and how it compares in speed to just using SSH to develop on the server.




can i use rclone on my local machine with vscode to edit a project on a server, which is tracked in git - are permissions, timestamps, symbolic links, working?

Try on a camp in ln19

Notes from trying on swelter.net:

Permissions are not preserved (at least by default). They appear to be set to 664 for normal files and 775 for directories
symbolic links are followed and mounted as normal directories
Timestamps actually do carry over, and seem to be converted to my system timezone 
If you're using the `-c` option for `ssh-add` (you're prompting for a confirmation), you have to confirm a comedically large amount

I logged into a personal server which lives in Germany, and built my Hugo static website I use to host a podcast. The page generation is very lightweight, but there are a lot of large files to copy. Doing this over SSH means I didn't have to install `hugo` on the server, which is nice because I've experienced some dependency problems on that server when trying to use Hugo.

This should be a perfect use for remote filesystem mounting, but copying files through my local machine comes at the heavy cost of time:

```
                      | EN  
-------------------+-----
  Pages            |  1  
  Paginator pages  |  0  
  Non-page files   |  0  
  Static files     |  9  
  Processed images |  0  
  Aliases          |  0  
  Sitemaps         |  1  
  Cleaned          |  0  

Total in 288560 ms
```

End Point site on ln19 via rclone mount:

```
                   |  EN
-------------------+-------
  Pages            | 2517
  Paginator pages  |  227
  Non-page files   | 2887
  Static files     |  527
  Processed images |    0
  Aliases          |  354
  Sitemaps         |    1
  Cleaned          |    0

Total in 8412837 ms
```
