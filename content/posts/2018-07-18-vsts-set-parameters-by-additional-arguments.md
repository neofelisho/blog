---
layout      : post
title       : "Set parameters by additional arguments of VSTS."
date        : 2018-07-18 23:00:00 +0800
categories  : programming
tags        : [Azure, VSTS, web.config, SetParameters, additional arguments, Continuous Delivery]
---
`Problem` There were some problems when we migrated our CI/CD from Git/Jenkins to VSTS.
After many trial-and-errors, we solved them and note here for memo.
- Incorrect replacement of connection strings.
- Incorrect replacement of email address.
- Incorrect replacement when using xml transform ```insert```.

`Summary`
- VSTS set parameters according to the base web.config.
- VSTS set parameters by replacing them in web.config.
- Following the previous two, there should be the config sections in base web.config we want to replace them.
We still can keep non-sensitve parameters in web.[environment].config, and it works with VSTS.
- Additional arguments is somethins like command line arguments, not XML format. 
Origianlly we set the email parameter in web.config like ```Neo Ho &lt;neofelisho@gmail.com&gt;```,
but in arguments we just use the plain text format like ````Neo Ho <neofelisho@gmail.com>```.