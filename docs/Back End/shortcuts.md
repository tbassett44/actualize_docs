---
title: Shortcuts
excerpt: In /bin Folder
deprecated: false
hidden: false
metadata:
  title: ''
  description: ''
  robots: index
next:
  description: ''
---
### Phi bin scripts — quick reference

A concise list of every script in bin/ with a short, code‑inferred description.

`actualize` — Fast‑forwards to origin/main, shows summary, then runs publish → admin modules → loaddb → perms.\
`admin` — Runs PHP CLI admin action via classes/admin\_cli.php (usage: admin \[args]).\
`backupdb2` — Dumps a MongoDB database using credentials in \~/priv/config\_auto.json (DB name placeholder XXXXX).\
`blame.sh` — Writes git blame for all tracked files to \_manage/data/blame.txt.\
`cdr` — cd into $\{ROOT}.\
`configurepostfix` — Applies postfix configuration via \_manage/postfix/load.php.\
`copyphpini` — Copies PHP cgi/cli ini templates (prod/dev) and reloads lighttpd config.\
`copysecrets` — Copies \~/priv to a remote host’s /var/www via scp (requires PEM and IP arg).\
`countsockets` — Prints the count of TCP sockets (wc -l /proc/net/tcp).\
`createpasscode` — Creates an AES‑256 key at \~/priv/aes\_key (and backups to /home/aes\_key) if absent.\
`db2` — Opens a Mongo shell using env credentials from \~/priv/config\_auto.json (auth DB admin).\
`debuglighttpd` — Validates lighttpd config (lighttpd -t and -tt).\
`downloadclip` — Creates a 1‑minute clip from a video URL using youtube-dl + ffmpeg (out.mkv).\
`enableipv6 `— Installs eth0 IPv6 config and prompts for reboot.\
`ensurecerts` — Moves \_certs into \~/priv and fixes secure permissions/ownership.\
`ensureffmpeg` — Installs ffmpeg from PPA (adds repo, updates, installs).\
`flower` — Generates cloc CSV for the repo into sites/flower/CodeFlower/data/app.cloc.\
`forever` — Wraps the forever CLI; special logging for notifier.js, otherwise generic control.\
`gita` — Fetches and hard‑resets to origin/main, then shows the commit summary.\
`initmongo` — Prepares /data/db and starts/enables mongod.\
`initpm2` — Restarts PM2 processes from ecosystem.config.js, saves, and sets startup.\
`installaws` — Installs Python 3.6 and awscli; sets python3 alternatives to 3.6.\
`installlighttpd` — Builds and installs lighttpd 1.4.76 from source; enables service.\
`installnode` — Installs Node.js 22.x via NodeSource setup script + apt.\
`installopenssl` — Builds and installs OpenSSL 3.2.0; restarts lighttpd.\
`installpuppeteer` — Installs system libraries required for Puppeteer/Chromium.\
`lighttpdconf` — Regenerates lighttpd.conf via \_manage/parselighttpd.php and reloads service.\
`loaddb` — Runs node/loaddb.js and \_manage/db2/loaddb.php (optional arg passthrough).\
`loadindex` — Runs node/loaddb.js with an argument (index build helper).\
`loadshortcodes` — Executes \_manage/loadshortcodes.php to register shortcodes.\
`loadshortcuts` — Copies all bin scripts to /usr/local/bin, chmod 0777; installs bashrc/aliases.\
`perms` — Recursively chmods $\{ROOT} to 0777 (very permissive; use with caution).\
`publish` — Runs \_manage/publish.php (passes up to two args).\
`renewssl` — Renews and updates SSL, then reloads lighttpd.\
`restoredb2` — Restores MongoDB dump to a specified Atlas cluster using creds from config.\
`runaws` — Invokes \_manage/aws.php with all passed arguments.\
`upgradelighttpd` — Builds and installs lighttpd 1.4.51 from source (older version).
