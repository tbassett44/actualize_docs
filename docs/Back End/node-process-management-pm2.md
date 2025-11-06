---
title: Node Process Management PM2
excerpt: ''
deprecated: false
hidden: false
metadata:
  title: ''
  description: ''
  robots: index
next:
  description: ''
---
[PM2 Website](https://pm2.keymetrics.io/) 

The following node scripts are run / managed by pm2, a process manager that ensures / monitors forever-running node scripts.  It detects if there is a process.exit and automatically restarts the script.  Currently, the following scripts run through it

* api2.js - node api that allows for advanced things like aggregation, searching, and other async needed api requests
* cron.js - where all functions that need to run on a regular basis go
* jobs.js - a queue system that ensures we can schedule jobs and not go past rate limits
* notifier.js - a lightweight system to poll DB and send email/push notifications to external gateways with rate limiting queues
* chat.io.js - a socket.io bridge for comments and other real time socket communications
* watcher.js - an intelligent file watching system that monitors pm2 and notifies admins if there is a continual error and supports emiting file updates to clients in dev environment for new code to be updated on the client
