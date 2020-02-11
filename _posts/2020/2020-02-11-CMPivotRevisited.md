---
id: 718
title: 'CM Pivot Revisited'
date: 2020-02-11
author: ryan2065@gmail.com
layout: post
guid: http://www.ephingadmin.com/?p=718
permalink: /CMPivotRevisited/
categories:
    - SCCM
    - CMPivot
    - MEMCM
---

# CM Pivot Revisited

I wanted to write a follow-up to my blog post [CMPivot Internals](https://www.ephingadmin.com/CMPivotInternals/) because in MEMCM 1910 CMPivot had a small, but huge change.

Now, after 1910, CMPivot processes queries client-side instead of server side!

![Huh](https://media.giphy.com/media/KFt2DA9T82paOA1Yci/giphy.gif)

Yeah, it's not obvious what that will do, so lets dive in!

## CMPivot is a Script!

Ok, *all* of CMPivot is not a script, but most of the code running on your client side computers is still a script. You can access this script still with the SQL query:

```SQL
SELECT CONVERT(varchar(max), Script) as 'ScriptText'
FROM Scripts
WHERE ScriptName = 'CMPivot'

```

Note: In SQL, if you want the full script information, you'll need to change your settings to let results have line breaks, and increase the max size of data returned.

Wait, increase the max size of data returned?!  Yeah, they embed a DLL inside the script now, so it's a lot longer. And that brings about the first change I want to talk about.

## Client Side Query Processing

Previously, CMPivot would process your Pivot queries server-side, tell the script client-side what data to gather, then sift through the data server-side and apply filters.  Because of how it worked, a lot of interesting limitations applied to CMPivot - the worst was you could only search through the last 50 or 100 lines of text files and Windows event logs.

Now that they embed the query parsing DLL in the PowerShell script, they parse the query client side and have adapted the script to use that information. Now if you say you want to search smsts.log for a specific line, it will search **ALL** of smsts.log instead of just the last 100 lines. It will also search through entire windows event logs for whatever you want to search for.

This makes CM Pivot SO much more powerful AND lighter on your server infrastructure.

## Comparing previous examples

If you look at the previous blog post, I showed how hard CMPivot could be on your environment by running the following query:

```Kestrel
CcmLog('Scripts') | where (Device == 'DeviceName') | order by DateTime desc | project Device, LogText, DateTime
```

Why was this query hard on your environment? If you pasted this into CMPivot and ran it, you'd see 0 results.  This is because no one really has a device named "DeviceName" in their environment. If you went to SQL though, and ran this query:

```SQL
SELECT [ResourceID]
      ,[ScriptOutput]
      ,[LastUpdateTime]
  FROM [vSMS_CMPivotResult]
  ORDER BY LastUpdateTime DESC
```

You'd see one result row for every device in your environment and in vSMS_CMPivotStatus you could even find the 50 lines of text from the Scripts.log file for every device. So that's a lot of data depending what collection you ran it on!

That process was all pre-1910.  Now, if I run the above query, 0 results are in vSMS_CMPivotResult because if you specify "DeviceName=x" it will only grab data from that specific device name! It's easy to say "duh, why didn't they do that before?!" but this comes with a trade-off.  If you want to "pivot" and suddenly want to do query all devices, it's going to have to create a new request and can't use cached results.  I think this change in process is 100% fine, but just know Pivot might *seem* slower now.

## Any areas of concern still?

This isn't a limitation of the technology, or a bug, but do watch out for database bloat and CMG costs. Each client can send a maximum amount of data at 128KB, which means every 8 clients can send 1MB of data that's stored temporarily in your database. Not huge numbers, but if you do a lot of querying of "Get me the entire contents of this log file" and then after that operation you filter to what you want, you could see issues with cost and database growth. Where I work, if I ran this CMPivot query against "All Systems" :

```
CcmLog('ccmexec') | order by DateTime desc | project Device, LogText, DateTime
```

I could expect the database to grow by about 50GB if all 400,000 machines are on. So any queries you need to run against 'All Systems' should probably first be fine-tuned against a smaller collection.

