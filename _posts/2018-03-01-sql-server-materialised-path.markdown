---
layout: post
title:  "SQL Server - Materialised path for fast parent child searches"
date:   2018-03-01 13:45:00 +0100
categories: sql server file-folder child-parent
disqus: true
live: false
---

Recently I was asked to develop a file sharing application that allows the creation of folder structures that are the same as the desktop. Much like a Dropbox ro Google Drive.

I was tasked to make this application with the use of SQL Server. For those who know SQL Server you may know there is a data type called the HierarchyId (2008+). Although this data tyoe is very usful it does have some issues when quering for all decendacy.

I found that at scale the HierarchyId wasn't good enough. Next I tried the use of recursive CTE's. Each row (file or folder) in the table would know about its parent via reference to its parents Id.

This brings me to the materialised path. A materialised path is a string representing the full path of the location of the row in relaton to all its anscentors. This is very similar to the HierarchyId. The benefit of a materialised view is the the deserilising the string will give you all ancestor nodes and a simply starts with LIKE statement will give you all the children. With the use of a simple index on the column the LIKE statement will casue an idex seek when quering for all children.