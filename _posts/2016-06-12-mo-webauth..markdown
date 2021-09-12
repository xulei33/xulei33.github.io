---
layout:     post
title:      "CI6.3 MO Web Authentication"
subtitle:   "MO Scheduler Web Auth"
date:       2016-06-12 21:00:00
author:     "Xulei"
header-img: "img/in-post/post-bg-re-vs-ng2.jpg"
header-mask: 0.3
catalog:    true
tags:
    - CI6x
    - Linux
---

> 
SAS Marketing Optimization Scheduling was designed to use the credentials of the user who is logged on to the client to connect to the scheduling service. These credentials were then added on the MOBatch -u and -p command-line options. If you used Integrated Windows Authentication or web authentication, there was no password for the credentials of a user who was logged on to the client. To enable scheduling from SAS Marketing Optimization, you must manually configure it.

## add MO properties

Create two properties on the MO Application Object by doing the following:

Invoke an interactive SAS session on the SAS tier machine.

Run MarketingOptimization_autoexec.

Enter and execute the following SAS code:

%mo_properties_edit_value(key = mktopt.metadata.schedule.user, value = moscheduser); (where this is a real host account)

%mo_properties_edit_value(key = mktopt.metadata.schedule.auth.domain, value = MOSchedAuthDomain);


## add MO Scheduling Group

In SAS Management Console, create the MO Scheduling Group with the following logon information:
Domain: MOSchedAuthDomain
User: moschedgroup
Password: password of moscheduser (real host account password)

## add MO Scheduler User

Also create the MO Scheduler User with the following logon information:
Domain: domain of IWA (or web authentication) 
User: moscheduser
Make this user a member of these groups: Marketing Optimization Administrators and Marketing Optimization Users.


## assign MO Scheduling user privilege

Add the MO Scheduling user to the Marketing Optimization Template in the Authorization Manager Access Control Templates with the permissions ReadMetadata and WriteMetadata.

## assign MO Scheduling user groups

Add the SAS System Services Group to the MO Scheduling Group.


## 修改SASServer6的serenv.sh或Wrapper.conf

> SASConfig/Lev1/Web/WebAppServer/SASServer6_1/bin/或SASConfig/Lev1/Web/WebAppServer/SASServer6_1/conf

> -Dsas.ci.using.Web.or.IWA=true

## Restart all servers

Restart the web application server for these changes to take effect.




