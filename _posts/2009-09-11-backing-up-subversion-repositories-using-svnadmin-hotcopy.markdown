---
date: '2009-09-11 12:18:02'
layout: post
slug: backing-up-subversion-repositories-using-svnadmin-hotcopy
status: publish
title: Backing Up Subversion Repositories Using svnadmin hotcopy
wordpress_id: '363'
categories:
- Development
- Infrastructure
tags:
- backup
- bash
- hotcopy
- subversion
- svn
- svnadmin
---

Just wanted to post this quick bash script to iterate over the repositories in a directory, perform an svnadmin hotcopy, and tar/gzip the output.

By using hotcopy this can be performed on a live subversion repository and will produce a pristine backup.

```bash
#!/bin/bash
REPOS_PATH=/var/repos
mkdir -p /backups/weekly
rm -rf /backups/tmp
mkdir -p /backups/tmp/repos
for i in $(ls $REPOS_PATH); do
        /path/to/svnadmin hotcopy $REPOS_PATH/$i /backups/tmp/repos/$i
done
FN=svn.weekly.`date '+%Y%m%d'`.tar.gz
tar -czf /backups/weekly/$FN -C /backups/tmp .
```

