---
id: 120
title: ConfigMgr 2012 Query for All Site Systems
date: 2014-08-05T21:56:46+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=120
permalink: /configmgr-2012-query-for-all-site-systems/
categories:
  - ConfigMgr
tags:
  - Query
  - Site systems
---
Did you ever want to create a collection with all ConfigMgr site servers? I like to create one for Endpoint Protection exceptions, but I want to to be a query so servers are automatically added. Here is the query I use:

select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System  WHERE ResourceNames[0] IN (Select Distinct ServerName FROM SMS_SystemResourceList)