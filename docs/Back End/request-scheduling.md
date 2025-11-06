---
title: Job Scheduling
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
A common pattern is the ability to run code asynchronously and returning the API request as soon as possible. An example of this might be triggering an email/push notification to be built and sent. 

```php
phi::scheduleJob('Unique ID',(int) timestamp of when to send,[options])
//EG
phi::scheduleJob($d['current']['id'].'_'.time(),time(),array(
  'url'=>phi::getSubdomain('api').'/core/module/ai_survey/process', // URL the request will send to
  'type'=>'url', //IF URL, it will 
  'data'=>array( //any relevant data to pass to the request.  This should be minimal.  Authorization protection is done via an internal API request with Admin token.
    'id'=>$d['current']['id']
  )
));
//phi::getSubdomain() is the environment specific way of getting the subdomain, eg if running on a dev server, you want the url to be like https://api-juicy.actualize.earth, not https://actualize.earth.  phi::getSubdomain() returns the correct subdomain depending on the server configuration
```

### Viewing Jobs in the Admin Panel
