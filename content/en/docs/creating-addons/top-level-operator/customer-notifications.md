---
title: "Customer Notifications"
date: 2022-12-15T00:53:51+01:00
---

There are multiple ways a user or group can get notified of service events (e.g.
planned maintenance, outages). There are two fields in the addon metadata file
(see Add-On metadata file [schema documentation](https://github.com/mt-sre/managed-tenants-cli/blob/main/docs/tenants/zz_metadata_schema_generated.md)
for more information) where email addresses can be provided:

* `addonOwner`: *REQUIRED* Point of contact for communications from Service Delivery to
  addon owners. Where possible, this should be a development team mailing list
  (rather than an individual developer).
* `addonNotifications`: This is a list of additional email addresses of
  employees who would like to receive notifications about a service.

There is also a mailing list that receives notifications for all services managed by Service Delivery.
Subscribe to the sd-notifications mailing list [here](https://post-office.corp.redhat.com/mailman/listinfo/sd-notifications).
