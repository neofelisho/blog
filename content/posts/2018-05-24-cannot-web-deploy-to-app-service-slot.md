---
title: "Cannot publish to deployment slot of Azure app service"
date: 2018-05-24T19:30:00+08:00
tags: [azure, web deploy, app service, deployment slot, publish]
category: "programming"
---

**Story**

![ERROR_DESTINATION_INVALID](https://pic.link/)

We occured `ERROR_DESTINATION_INVALID` error while we using web deploy to publish web site to deployment slot of Azure app service.

**Solution**
Because of the site was running, some files were locked by server. Stop the app service (or iis) and re-deploy it again.