---
layout:     post
title:      "CI6.3 MO Web Authentication"
subtitle:   "SAS/Access DB2"
date:       2016-06-13 21:00:00
author:     "Xulei"
header-img: "img/in-post/post-bg-re-vs-ng2.jpg"
header-mask: 0.3
catalog:    true
tags:
    - CI6x
    - Linux
---


SAS Marketing Optimization Scheduling was designed to use the credentials of the user that is logged on to the client to connect to the scheduling service and then add these credentials on the MOBatch -u and -p command-line options. If you used Integrated Windows authentication or web authentication, there was no password for the credentials of a user that was logged on to the client. Therefore, it was not possible to schedule from SAS Marketing Optimization.

Click the Hot Fix tab in this note to access the hot fix for this issue. This hot fix makes it possible to schedule if you use Integrated Windows authentication or web authentication. The hot fix adds functionality so that a single host user account can be configured to connect to the scheduler and run all scheduled operations. 

After you apply the hot fix, it is necessary to make the followings changes:
1.Create two properties on the MO Application Object:
a.Invoke an interactive SAS® session on the SAS tier machine.
b.Run Marketing_Optimization_autoexec.
c.Enter and execute the following SAS code:
%mo_properties_edit_value(key = mktopt.metadata.schedule.user, value = moscheduser);  (where this is a real host account)
%mo_properties_edit_value(key = mktopt.metadata.schedule.auth.domain, value = MOSchedAuthDomain);  

2.In SAS® Management Console, create the MO Scheduling Group with the following logon information:

Domain: MOSchedAuthDomain
User: dummy
Password: password of moscheduser (real host account password)

3.Also create a MO Schedule User with the following logon information:

Domain: domain of IWA
User: moscheduser

Make this user a member of these groups: Marketing Optimization Administrators; Marketing Optimization Report Consumers; and Marketing Optimization Users.

4.Add the MO Scheduling User to the Marketing Optimization Template in the Authorization Manager Access Control Templates with the permissions ReadMetadata, WriteMetadata.



5.Add the SAS System Services Group to the MO Scheduling Group.



6.Restart the web application server for these changes to take effect.



