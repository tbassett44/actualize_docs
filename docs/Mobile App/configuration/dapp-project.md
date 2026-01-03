---
title: DAPP Project
deprecated: false
hidden: false
metadata:
  robots: index
---
To register your project in the DAPP ecosystem, you need to add a file to register your project

```
/dapp/config/[project_id].json
```

Within that File, you just need to register the directories that will be used in the build process

```json
{
	"name":"Actualize",
	"dirs":{
		"project":"/Volumes/Projects/actualize",
		"template":"/Volumes/Projects/_templates/actualize",
		"babies":"/Volumes/Projects/babies",
		"backupdir":"/Volumes/Projects/babies/backups"
	}
}
```

<br />
