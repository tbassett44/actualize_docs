---
title: Common Tools (phi.php)
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
# `phi.php` Utility Functions

*Auto-generated overview. Descriptions are inferred heuristically from names and code cues—please review.*

## Index

* [breakNetwork](#breaknetwork)
* [flagIp](#flagip)
* [debug](#debug)
* [stopMaliciousIp](#stopmaliciousip)
* [isVPN](#isvpn)
* [digestApi](#digestapi)
* [logRequest](#logrequest)
* [convertToPlainText](#converttoplaintext)
* [getUrlInfo](#geturlinfo)
* [ordinal](#ordinal)
* [getEmailToken](#getemailtoken)
* [formatMoney](#formatmoney)
* [getModel](#getmodel)
* [getLocationAddress](#getlocationaddress)
* [getLocationInfo](#getlocationinfo)
* [getCoords](#getcoords)
* [getLatestFileCount](#getlatestfilecount)
* [getLineCount](#getlinecount)
* [buildSearchQuery](#buildsearchquery)
* [ensure](#ensure)
* [getMaxBound](#getmaxbound)
* [getMapboxPolygon](#getmapboxpolygon)
* [getRandomSplash](#getrandomsplash)
* [getRand](#getrand)
* [gzip](#gzip)
* [prettyfyTag](#prettyfytag)
* [obfuscateText](#obfuscatetext)
* [throttleJob](#throttlejob)
* [rateLimitMessage](#ratelimitmessage)
* [emitHook](#emithook)
* [saveHooks](#savehooks)
* [fixContent](#fixcontent)
* [scheduleBulkJob](#schedulebulkjob)
* [saveBulkJobs](#savebulkjobs)
* [scheduleJob](#schedulejob)
* [clearJob](#clearjob)
* [saveBase64File](#savebase64file)
* [stripJsonComments](#stripjsoncomments)
* [removeJob](#removejob)
* [dieMessage](#diemessage)
* [die404](#die404)
* [getAppList](#getapplist)
* [getColor](#getcolor)
* [prompt](#prompt)
* [promptSilent](#promptsilent)
* [getCreds](#getcreds)
* [getApp](#getapp)
* [fixHtmlContent](#fixhtmlcontent)
* [mongoRand](#mongorand)
* [getMongoId](#getmongoid)
* [isValidApp](#isvalidapp)
* [getMongoTS](#getmongots)
* [getAnonToken](#getanontoken)
* [tagToName](#tagtoname)
* [getMagicLink](#getmagiclink)
* [getMagicLinkId](#getmagiclinkid)
* [makeMongoTime](#makemongotime)
* [getKeyWords](#getkeywords)
* [strpos\_array](#strpos_array)
* [extract](#extract)
* [get\_server\_memory\_usage](#get_server_memory_usage)
* [cpuStats](#cpustats)
* [getCoordsFromIp](#getcoordsfromip)
* [isAllowed](#isallowed)
* [isScrape](#isscrape)
* [isBot](#isbot)
* [hasScope](#hasscope)
* [registerToApp](#registertoapp)
* [getImageAR](#getimagear)
* [makeUrl](#makeurl)
* [getReplayUrl](#getreplayurl)
* [getQS](#getqs)
* [getUAInfo](#getuainfo)
* [isok](#isok)
* [downloadSite](#downloadsite)
* [parseObject](#parseobject)
* [parseString](#parsestring)
* [dotGet](#dotget)
* [dotPush](#dotpush)
* [dotSet](#dotset)
* [dotUnset](#dotunset)
* [getTimeZones](#gettimezones)
* [makeLinkUid](#makelinkuid)
* [getDB](#getdb)
* [flog](#flog)
* [getSecurityInfo](#getsecurityinfo)
* [log](#log)
* [dieMongo](#diemongo)
* [alertAdmin](#alertadmin)
* [execNode](#execnode)
* [execNode2](#execnode2)
* [clearStdin](#clearstdin)
* [clog](#clog)
* [redir](#redir)
* [getIP](#getip)
* [outputFileToHeaders](#outputfiletoheaders)
* [isValidUrl](#isvalidurl)
* [XMLtoJSON](#xmltojson)
* [niceGUID](#niceguid)
* [sanitize](#sanitize)
* [strToHex](#strtohex)
* [hexToStr](#hextostr)
* [genCode](#gencode)
* [outputCSV](#outputcsv)
* [downloadFile](#downloadfile)
* [download](#download)
* [deferTask](#defertask)
* [getMonthDiff](#getmonthdiff)
* [execTasks](#exectasks)
* [toMoney](#tomoney)
* [saveToMongo](#savetomongo)
* [fixImg](#fiximg)
* [clean](#clean)
* [minifile](#minifile)
* [getPageData](#getpagedata)
* [translate](#translate)
* [translateBlock](#translateblock)
* [translateSRT](#translatesrt)
* [getSES](#getses)
* [getAwsCreds](#getawscreds)
* [getTranscribe](#gettranscribe)
* [getTranslate](#gettranslate)
* [getS3](#gets3)
* [getGlacier](#getglacier)
* [deleteS3dir](#deletes3dir)
* [getPrefix](#getprefix)
* [toURL](#tourl)
* [keepFields](#keepfields)
* [formatTime](#formattime)
* [getFirstName](#getfirstname)
* [getLastName](#getlastname)
* [objectToSize](#objecttosize)
* [getCollectionSize](#getcollectionsize)
* [formatBytes](#formatbytes)
* [renderRedactorContent](#renderredactorcontent)
* [processVars](#processvars)
* [render2](#render2)
* [createElementFromHTML](#createelementfromhtml)
* [convertArray](#convertarray)
* [render](#render)
* [getSubdomain](#getsubdomain)
* [clearFileCache](#clearfilecache)
* [clearCache](#clearcache)
* [cache](#cache)
* [uncache](#uncache)
* [getRemainingTime](#getremainingtime)
* [getWeather](#getweather)
* [getWindDirection](#getwinddirection)
* [time](#time)
* [getForecast](#getforecast)
* [push](#push)
* [getBackTrace](#getbacktrace)
* [sendPush](#sendpush)
* [readLogFile](#readlogfile)
* [formatNumber](#formatnumber)
* [getLocalIp](#getlocalip)
* [getAesKey](#getaeskey)
* [encryptFile](#encryptfile)
* [decryptFile](#decryptfile)
* [aesEncryptFile](#aesencryptfile)
* [aesDecryptFile](#aesdecryptfile)
* [encrypt](#encrypt)
* [decrypt](#decrypt)
* [safeEncrypt](#safeencrypt)
* [safeDecrypt](#safedecrypt)
* [getPostData](#getpostdata)
* [cleanData](#cleandata)
* [decodeUnicode](#decodeunicode)
* [isRealObject](#isrealobject)
* [curl](#curl)
* [getIndexByKey](#getindexbykey)
* [isJson](#isjson)
* [generateCallTrace](#generatecalltrace)
* [getUniqueNumber](#getuniquenumber)
* [getUniqueTime](#getuniquetime)
* [mail](#mail)
* [sendMail](#sendmail)
* [isValidEmail](#isvalidemail)
* [sendRawMail](#sendrawmail)
* [getObjectKeys](#getobjectkeys)
* [getObjectKeys2](#getobjectkeys2)
* [db](#db)
* [getImg](#getimg)
* [isadmin](#isadmin)
* [getDay](#getday)
* [uploadProgress](#uploadprogress)
* [relTime](#reltime)
* [getFiles](#getfiles)
* [getDirs](#getdirs)
* [loadProxyImage](#loadproxyimage)
* [get\_headers\_from\_curl\_response](#get_headers_from_curl_response)
* [getUrlsFromString](#geturlsfromstring)
* [getUrlHeaders](#geturlheaders)
* [limitLength](#limitlength)
* [getPageTags](#getpagetags)
* [getBase64Image](#getbase64image)
* [getBase64data](#getbase64data)
* [getFileInfo](#getfileinfo)
* [getPath](#getpath)
* [getDomain](#getdomain)
* [sort](#sort)
* [getIcons](#geticons)
* [fixFontCss](#fixfontcss)
* [getFont64](#getfont64)
* [get\_timezone\_offset](#get_timezone_offset)
* [getFileConfs](#getfileconfs)
* [publish](#publish)
* [renderFileTemplate](#renderfiletemplate)
* [renderTemplate](#rendertemplate)
* [deleteDirectory](#deletedirectory)
* [loadCsvData](#loadcsvdata)
* [copyDirectory](#copydirectory)
* [getByKey](#getbykey)
* [clear\_tags\_redactor](#clear_tags_redactor)
* [rateLimit](#ratelimit)
* [decryptCode](#decryptcode)
* [getPurifier](#getpurifier)
* [ensureRedactorContent](#ensureredactorcontent)
* [cleanRedactorContent](#cleanredactorcontent)
* [clear\_tags](#clear_tags)
* [isVideo](#isvideo)
* [mime\_content\_type](#mime_content_type)
* [isWebsite](#iswebsite)
* [getContentType](#getcontenttype)
* [upload](#upload)
* [exportCSV](#exportcsv)
* [flatten](#flatten)
* [getDistanceBetweenPoints](#getdistancebetweenpoints)
* [getLatLngDistance](#getlatlngdistance)
* [getDiff](#getdiff)
* [exportjson](#exportjson)
* [diejson](#diejson)

***

### breakNetwork

**Signature:** `breakNetwork()`

**What it likely does:** Provide breaknetwork. likely involves: ip/network, json response, database.

***

### flagIp

**Signature:** `flagIp()`

**What it likely does:** Flag flagip. likely involves: ip/network, json response, database.

***

### debug

**Signature:** `debug($obj)`

**What it likely does:** Provide debug. likely involves: ip/network, json response, database.

***

### stopMaliciousIp

**Signature:** `stopMaliciousIp()`

**What it likely does:** Provide stopmaliciousip. likely involves: ip/network, database.

***

### isVPN

**Signature:** `isVPN($ip=false,$nocache=false)`

**What it likely does:** Check whether isvpn. likely involves: http request, ip/network, vpn check, cache, database.

***

### digestApi

**Signature:** `digestApi($module,$path)`

**What it likely does:** Provide digestapi.

**Inline comment found:**

> //bad request or didnt get data...dont block!

***

### logRequest

**Signature:** `logRequest($r,$out)`

**What it likely does:** Provide logrequest. likely involves: ip/network, logging.

***

### convertToPlainText

**Signature:** `convertToPlainText($html)`

**What it likely does:** Convert converttoplaintext. likely involves: ip/network.

***

### getUrlInfo

**Signature:** `getUrlInfo($r,$return_metadata)`

**What it likely does:** Get geturlinfo. likely involves: database.

***

### ordinal

**Signature:** `ordinal($number)`

**What it likely does:** Provide ordinal. likely involves: email.

***

### getEmailToken

**Signature:** `getEmailToken($uid)`

**What it likely does:** Get getemailtoken. likely involves: token.

***

### formatMoney

**Signature:** `formatMoney($amt)`

**What it likely does:** Format formatmoney.

***

### getModel

**Signature:** `getModel($model,$site=false)`

**What it likely does:** Get getmodel.

***

### getLocationAddress

**Signature:** `getLocationAddress($location,$link=false)`

**What it likely does:** Get getlocationaddress.

***

### getLocationInfo

**Signature:** `getLocationInfo($loc,$force=false,$update=true)`

**What it likely does:** Get getlocationinfo. likely involves: database.

**Inline comment found:**

> //mapbox location

***

### getCoords

**Signature:** `getCoords($id)`

**What it likely does:** Get getcoords. likely involves: http request.

***

### getLatestFileCount

**Signature:** `getLatestFileCount()`

**What it likely does:** Get getlatestfilecount.

***

### getLineCount

**Signature:** `getLineCount($file)`

**What it likely does:** Get getlinecount.

***

### buildSearchQuery

**Signature:** `buildSearchQuery($q,$search,$lookin)`

**What it likely does:** Build buildsearchquery.

***

### ensure

**Signature:** `ensure($r,$fields,$dieerror=true,$perms=false,$roles=false)`

**What it likely does:** Ensure ensure.

**Inline comment found:**

> // $q\['$or']=array('data.title'=>$regex);

***

### getMaxBound

**Signature:** `getMaxBound($val,$type)`

**What it likely does:** Get getmaxbound.

***

### getMapboxPolygon

**Signature:** `getMapboxPolygon($bounds)`

**What it likely does:** Get getmapboxpolygon.

***

### getRandomSplash

**Signature:** `getRandomSplash($album,$retry=5)`

**What it likely does:** Get getrandomsplash.

***

### getRand

**Signature:** `getRand()`

**What it likely does:** Get getrand. likely involves: ip/network, zip archive.

**Inline comment found:**

> //should be recursive till it works, but dont let go forever..

***

### gzip

**Signature:** `gzip($contents)`

**What it likely does:** Provide gzip.

***

### prettyfyTag

**Signature:** `prettyfyTag($val)`

**What it likely does:** Provide prettyfytag.

***

### obfuscateText

**Signature:** `obfuscateText($text)`

**What it likely does:** Provide obfuscatetext. likely involves: database.

***

### throttleJob

**Signature:** `throttleJob($id,$job,$max_ts,$queue_opts=[],$force=false,$debug=false)`

**What it likely does:** Provide throttlejob. likely involves: database.

***

### rateLimitMessage

**Signature:** `rateLimitMessage($id,$max)`

**What it likely does:** Provide ratelimitmessage. likely involves: database.

***

### emitHook

**Signature:** `emitHook($app,$when,$opts,$return=false)`

**What it likely does:** Provide emithook. likely involves: database.

***

### saveHooks

**Signature:** `saveHooks($hooks)`

**What it likely does:** Save savehooks. likely involves: database.

***

### fixContent

**Signature:** `fixContent($content,$style=false)`

**What it likely does:** Provide fixcontent.

**Inline comment found:**

> //clean

***

### scheduleBulkJob

**Signature:** `scheduleBulkJob($id,$when,$opts,$queue_opts=[])`

**What it likely does:** Provide schedulebulkjob.

**Inline comment found:**

> //redactor content, already formatted

***

### saveBulkJobs

**Signature:** `saveBulkJobs($hooks)`

**What it likely does:** Save savebulkjobs. likely involves: database.

***

### scheduleJob

**Signature:** `scheduleJob($id,$when,$opts,$queue_opts=[],$update=false)`

**What it likely does:** Provide schedulejob.

***

### clearJob

**Signature:** `clearJob($id)`

**What it likely does:** Provide clearjob. likely involves: database, image.

***

### saveBase64File

**Signature:** `saveBase64File($base64_string, $output_file)`

**What it likely does:** Save file. likely involves: image.

***

### stripJsonComments

**Signature:** `stripJsonComments($json)`

**What it likely does:** Provide stripjsoncomments. likely involves: ip/network.

**Inline comment found:**

> // clean up the file resource

***

### removeJob

**Signature:** `removeJob($id)`

**What it likely does:** Provide removejob.

***

### dieMessage

**Signature:** `dieMessage($type=404,$message='Page Not Found')`

**What it likely does:** Provide diemessage.

***

### die404

**Signature:** `die404($simple=false)`

**What it likely does:** Provide die404. likely involves: database.

***

### getAppList

**Signature:** `getAppList()`

**What it likely does:** Get getapplist. likely involves: database.

***

### getColor

**Signature:** `getColor($index=false)`

**What it likely does:** Get getcolor.

***

### prompt

**Signature:** `prompt($prompt = "Prompt:",$answers=false)`

**What it likely does:** Provide prompt.

***

### promptSilent

**Signature:** `promptSilent($prompt = "Enter Password:")`

**What it likely does:** Provide promptsilent. likely involves: ip/network.

***

### getCreds

**Signature:** `getCreds($app,$site,$user)`

**What it likely does:** Get getcreds. likely involves: cache, database.

***

### getApp

**Signature:** `getApp($id)`

**What it likely does:** Get getapp. likely involves: cache, database.

***

### fixHtmlContent

**Signature:** `fixHtmlContent($content)`

**What it likely does:** Provide fixhtmlcontent.

**Inline comment found:**

> //eventually use memcached/db

***

### mongoRand

**Signature:** `mongoRand($max)`

**What it likely does:** Provide mongorand.

**Inline comment found:**

> // 	}else\{

***

### getMongoId

**Signature:** `getMongoId($ts)`

**What it likely does:** Get getmongoid.

**Inline comment found:**

> // $maxl=strlen((string) $max)-1;

***

### isValidApp

**Signature:** `isValidApp($id,$secret)`

**What it likely does:** Check whether isvalidapp. likely involves: database.

***

### getMongoTS

**Signature:** `getMongoTS($t)`

**What it likely does:** Get getmongots. likely involves: database.

***

### getAnonToken

**Signature:** `getAnonToken($app)`

**What it likely does:** Get getanontoken. likely involves: database, token.

***

### tagToName

**Signature:** `tagToName($tag)`

**What it likely does:** Provide tagtoname. likely involves: database.

**Inline comment found:**

> //1 hour max!

***

### getMagicLink

**Signature:** `getMagicLink($app_id)`

**What it likely does:** Get getmagiclink. likely involves: database.

***

### getMagicLinkId

**Signature:** `getMagicLinkId($app,$user,$uuid,$source,$appid)`

**What it likely does:** Get getmagiclinkid. likely involves: ip/network, database.

**Inline comment found:**

> //return';

***

### makeMongoTime

**Signature:** `makeMongoTime($ts)`

**What it likely does:** Provide makemongotime.

***

### getKeyWords

**Signature:** `getKeyWords($message,$maxLength=600)`

**What it likely does:** Get getkeywords.

***

### strpos\_array

**Signature:** `strpos_array($haystack, $needles, &$str_return)`

**What it likely does:** Provide array.

***

### extract

**Signature:** `extract($message,$max=6,$min_count=2)`

**What it likely does:** Provide extract. likely involves: database, object storage.

***

### get\_server\_memory\_usage

**Signature:** `get_server_memory_usage()`

**What it likely does:** Get server memory usage.

***

### cpuStats

**Signature:** `cpuStats()`

**What it likely does:** Provide cpustats.

***

### getCoordsFromIp

**Signature:** `getCoordsFromIp($ip=false,$clean=false)`

**What it likely does:** Get getcoordsfromip. likely involves: ip/network, database.

***

### isAllowed

**Signature:** `isAllowed($list,$uri=false)`

**What it likely does:** Check whether isallowed.

***

### isScrape

**Signature:** `isScrape($force=false)`

**What it likely does:** Check whether isscrape.

***

### isBot

**Signature:** `isBot()`

**What it likely does:** Check whether isbot.

***

### hasScope

**Signature:** `hasScope($has,$scopes)`

**What it likely does:** Determine if hasscope.

**Inline comment found:**

> // 'Above given bots detected'

***

### registerToApp

**Signature:** `registerToApp($uid,$appid,$token=false,$scopes=false,$temporary=false,$user=false)`

**What it likely does:** Provide registertoapp.

***

### getImageAR

**Signature:** `getImageAR($path)`

**What it likely does:** Get getimagear. likely involves: image.

***

### makeUrl

**Signature:** `makeUrl($r=false)`

**What it likely does:** Provide makeurl. likely involves: ip/network.

***

### getReplayUrl

**Signature:** `getReplayUrl($qs_only=false)`

**What it likely does:** Get getreplayurl.

**Inline comment found:**

> // phi::log($\_SERVER);

***

### getQS

**Signature:** `getQS($dont_include=array()`

**What it likely does:** Get getqs.

**Inline comment found:**

> //$tqs='\_base64='.base64\_encode(json\_encode($qs));

***

### getUAInfo

**Signature:** `getUAInfo()`

**What it likely does:** Get getuainfo.

**Inline comment found:**

> //could include other thigns!

***

### isok

**Signature:** `isok()`

**What it likely does:** Check whether isok. likely involves: http request.

***

### downloadSite

**Signature:** `downloadSite($url)`

**What it likely does:** Download downloadsite. likely involves: http request.

***

### parseObject

**Signature:** `parseObject($object,$data)`

**What it likely does:** Parse parseobject.

**Inline comment found:**

> //get index from wayback machine

***

### parseString

**Signature:** `parseString($string,$data)`

**What it likely does:** Parse parsestring.

***

### dotGet

**Signature:** `dotGet($key,$doc,$falsey=false)`

**What it likely does:** Provide dotget.

***

### dotPush

**Signature:** `dotPush($key,$doc,$val,$to=false)`

**What it likely does:** Provide dotpush.

**Inline comment found:**

> //

***

### dotSet

**Signature:** `dotSet($key,$doc,$val,$to=false)`

**What it likely does:** Provide dotset.

***

### dotUnset

**Signature:** `dotUnset($key,$doc)`

**What it likely does:** Provide dotunset.

***

### getTimeZones

**Signature:** `getTimeZones()`

**What it likely does:** Get gettimezones. likely involves: ip/network.

**Inline comment found:**

> //die(json\_encode($doc));

***

### makeLinkUid

**Signature:** `makeLinkUid($url)`

**What it likely does:** Provide makelinkuid.

***

### getDB

**Signature:** `getDB($writeable=false,$db=false,$admin=false)`

**What it likely does:** Get getdb.

***

### flog

**Signature:** `flog($msg)`

**What it likely does:** Provide flog. likely involves: logging.

**Inline comment found:**

> // connect

***

### getSecurityInfo

**Signature:** `getSecurityInfo()`

**What it likely does:** Get getsecurityinfo. likely involves: ip/network.

***

### log

**Signature:** `log($msg,$type='default',$error=false)`

**What it likely does:** Provide log. likely involves: ip/network, logging.

***

### dieMongo

**Signature:** `dieMongo($obj)`

**What it likely does:** Provide diemongo. likely involves: ip/network, json response, database, email.

***

### alertAdmin

**Signature:** `alertAdmin($message,$maxrate=false,$otheremails=false)`

**What it likely does:** Alert alertadmin. likely involves: database.

**Inline comment found:**

> //maxrate in minutes

***

### execNode

**Signature:** `execNode($exec,$base64=false,$debug=false)`

**What it likely does:** Provide execnode. likely involves: ip/network, image.

***

### execNode2

**Signature:** `execNode2($scriptstpl,$data,$debug=false)`

**What it likely does:** Provide execnode2. likely involves: ip/network, image.

***

### clearStdin

**Signature:** `clearStdin()`

**What it likely does:** Provide clearstdin. likely involves: json response.

**Inline comment found:**

> //die($exec);

***

### clog

**Signature:** `clog($m,$f=1)`

**What it likely does:** Provide clog. likely involves: ip/network, json response, logging.

**Inline comment found:**

> //die($exec);

***

### redir

**Signature:** `redir($location)`

**What it likely does:** Provide redir. likely involves: ip/network.

**Inline comment found:**

> //replace line

***

### getIP

**Signature:** `getIP()`

**What it likely does:** Get getip. likely involves: ip/network.

***

### outputFileToHeaders

**Signature:** `outputFileToHeaders($o)`

**What it likely does:** Provide outputfiletoheaders.

***

### isValidUrl

**Signature:** `isValidUrl($url)`

**What it likely does:** Check whether isvalidurl.

***

### XMLtoJSON

**Signature:** `XMLtoJSON($url)`

**What it likely does:** Provide xmltojson.

**Inline comment found:**

> //only allow text/html pages

***

### niceGUID

**Signature:** `niceGUID($oopts)`

**What it likely does:** Provide niceguid.

***

### sanitize

**Signature:** `sanitize($name)`

**What it likely does:** Provide sanitize.

**Inline comment found:**

> //will ensure unique!

***

### strToHex

**Signature:** `strToHex($string)`

**What it likely does:** Provide strtohex.

***

### hexToStr

**Signature:** `hexToStr($hex)`

**What it likely does:** Provide hextostr.

***

### genCode

**Signature:** `genCode($len=4)`

**What it likely does:** Provide gencode.

***

### outputCSV

**Signature:** `outputCSV($data)`

**What it likely does:** Provide outputcsv.

***

### downloadFile

**Signature:** `downloadFile($src,$mime,$name,$isText=false)`

**What it likely does:** Download downloadfile. likely involves: ip/network.

**Inline comment found:**

> //output", "wb");

***

### download

**Signature:** `download($r)`

**What it likely does:** Download download. likely involves: image.

***

### deferTask

**Signature:** `deferTask($tasks)`

**What it likely does:** Provide defertask.

***

### getMonthDiff

**Signature:** `getMonthDiff($start,$end)`

**What it likely does:** Get getmonthdiff.

***

### execTasks

**Signature:** `execTasks($tasks,$debug=false)`

**What it likely does:** Provide exectasks.

**Inline comment found:**

> // }

***

### toMoney

**Signature:** `toMoney($amount)`

**What it likely does:** Provide tomoney.

**Inline comment found:**

> //debug mode

***

### saveToMongo

**Signature:** `saveToMongo($data,$prefix='')`

**What it likely does:** Save savetomongo.

***

### fixImg

**Signature:** `fixImg($src,$host,$imgpath)`

**What it likely does:** Provide fiximg.

***

### clean

**Signature:** `clean($string)`

**What it likely does:** Provide clean.

**Inline comment found:**

> //')===false)\{

***

### minifile

**Signature:** `minifile($root,$files)`

**What it likely does:** Provide minifile.

**Inline comment found:**

> //'.$host.$src;

***

### getPageData

**Signature:** `getPageData($url)`

**What it likely does:** Get getpagedata.

**Inline comment found:**

> //remove comments

***

### translate

**Signature:** `translate($from_lang,$to_lang,$content)`

**What it likely does:** Provide translate.

***

### translateBlock

**Signature:** `translateBlock($from_lang,$to_lang,$content)`

**What it likely does:** Provide translateblock.

**Inline comment found:**

> // REQUIRED

***

### translateSRT

**Signature:** `translateSRT($from_lang,$to_lang,$content)`

**What it likely does:** Provide translatesrt.

**Inline comment found:**

> //first fragment will always be ""

***

### getSES

**Signature:** `getSES()`

**What it likely does:** Get getses.

**Inline comment found:**

> //die(json\_encode($info,JSON\_PRETTY\_PRINT));

***

### getAwsCreds

**Signature:** `getAwsCreds()`

**What it likely does:** Get getawscreds.

**Inline comment found:**

> //other credss

***

### getTranscribe

**Signature:** `getTranscribe()`

**What it likely does:** Get gettranscribe.

***

### getTranslate

**Signature:** `getTranslate()`

**What it likely does:** Get gettranslate. likely involves: object storage.

***

### getS3

**Signature:** `getS3()`

**What it likely does:** Get gets3. likely involves: object storage.

***

### getGlacier

**Signature:** `getGlacier()`

**What it likely does:** Get getglacier. likely involves: object storage.

***

### deleteS3dir

**Signature:** `deleteS3dir($bucket,$dir=false)`

**What it likely does:** Delete dir. likely involves: object storage.

***

### getPrefix

**Signature:** `getPrefix()`

**What it likely does:** Get getprefix.

***

### toURL

**Signature:** `toURL($str)`

**What it likely does:** Provide tourl.

**Inline comment found:**

> //';

***

### keepFields

**Signature:** `keepFields($arr,$fields=false)`

**What it likely does:** Provide keepfields.

**Inline comment found:**

> //';

***

### formatTime

**Signature:** `formatTime($ts,$type='',$et=false,$tz='America/Denver',$addtz=true)`

**What it likely does:** Format formattime.

***

### getFirstName

**Signature:** `getFirstName($name)`

**What it likely does:** Get getfirstname.

***

### getLastName

**Signature:** `getLastName($name)`

**What it likely does:** Get getlastname.

***

### objectToSize

**Signature:** `objectToSize($object,$number=false)`

**What it likely does:** Provide objecttosize.

***

### getCollectionSize

**Signature:** `getCollectionSize($coll)`

**What it likely does:** Get getcollectionsize.

***

### formatBytes

**Signature:** `formatBytes($bytes, $precision = 2)`

**What it likely does:** Format formatbytes.

***

### renderRedactorContent

**Signature:** `renderRedactorContent($content,$data)`

**What it likely does:** Render renderredactorcontent.

**Inline comment found:**

> // Uncomment one of the following alternatives

***

### processVars

**Signature:** `processVars($vars)`

**What it likely does:** Provide processvars.

***

### render2

**Signature:** `render2($opts)`

**What it likely does:** Render render2. likely involves: email.

**Inline comment found:**

> //die(json\_encode($vars\[$k]));

***

### createElementFromHTML

**Signature:** `createElementFromHTML($dom,$str,$containerTag)`

**What it likely does:** Create createelementfromhtml.

**Inline comment found:**

> //die($em);

***

### convertArray

**Signature:** `convertArray($arr, $narr = array()`

**What it likely does:** Convert convertarray.

***

### render

**Signature:** `render($opts)`

**What it likely does:** Render render.

***

### getSubdomain

**Signature:** `getSubdomain($type,$no_protocol=false,$force_prod=false)`

**What it likely does:** Get getsubdomain.

**Inline comment found:**

> //assume failed

***

### clearFileCache

**Signature:** `clearFileCache($types,$internal=1,$force=false)`

**What it likely does:** Provide clearfilecache. likely involves: cache.

***

### clearCache

**Signature:** `clearCache($filename,$force=false,$internal=true,$slave=false)`

**What it likely does:** Provide clearcache. likely involves: cache.

***

### cache

**Signature:** `cache($filename,$loaddata,$force=false,$json=false)`

**What it likely does:** Cache cache. likely involves: cache.

**Inline comment found:**

> //ensure others in system can write/clear lock

***

### uncache

**Signature:** `uncache($filename)`

**What it likely does:** Provide uncache. likely involves: cache.

**Inline comment found:**

> //ensure everyone can edit/delete

***

### getRemainingTime

**Signature:** `getRemainingTime($elapsed,$progress)`

**What it likely does:** Get getremainingtime.

***

### getWeather

**Signature:** `getWeather($opts,$force=false)`

**What it likely does:** Get getweather. likely involves: cache.

***

### getWindDirection

**Signature:** `getWindDirection($deg)`

**What it likely does:** Get getwinddirection.

***

### time

**Signature:** `time($name,$maxlog=false)`

**What it likely does:** Provide time. likely involves: ip/network.

***

### getForecast

**Signature:** `getForecast($data)`

**What it likely does:** Get getforecast. likely involves: ip/network, push notifications.

***

### push

**Signature:** `push($uid,$channel,$data)`

**What it likely does:** Provide push. likely involves: http request.

***

### getBackTrace

**Signature:** `getBackTrace()`

**What it likely does:** Get getbacktrace.

***

### sendPush

**Signature:** `sendPush($to,$message,$intent='',$count=1,$sound='',$title='',$data='',$to_uid=false)`

**What it likely does:** Send sendpush. likely involves: database, email, push notifications.

***

### readLogFile

**Signature:** `readLogFile($file,$maxPer,$gotLinesFunction,$opts)`

**What it likely does:** Provide readlogfile. likely involves: logging.

**Inline comment found:**

> //web

***

### formatNumber

**Signature:** `formatNumber($n)`

**What it likely does:** Format formatnumber. likely involves: http request, ip/network.

**Inline comment found:**

> // error opening the file.

***

### getLocalIp

**Signature:** `getLocalIp()`

**What it likely does:** Get getlocalip. likely involves: http request, ip/network.

**Inline comment found:**

> // error opening the file.

***

### getAesKey

**Signature:** `getAesKey($type='iv')`

**What it likely does:** Get getaeskey.

**Inline comment found:**

> //169.254.169.254/latest/meta-data/local-ipv4');

***

### encryptFile

**Signature:** `encryptFile($source, $dest)`

**What it likely does:** Encrypt encryptfile. likely involves: crypto.

***

### decryptFile

**Signature:** `decryptFile($source, $dest)`

**What it likely does:** Decrypt decryptfile.

***

### aesEncryptFile

**Signature:** `aesEncryptFile($content)`

**What it likely does:** Provide aesencryptfile. likely involves: ip/network.

***

### aesDecryptFile

**Signature:** `aesDecryptFile($content)`

**What it likely does:** Provide aesdecryptfile. likely involves: ip/network.

***

### encrypt

**Signature:** `encrypt($data)`

**What it likely does:** Encrypt encrypt. likely involves: crypto.

**Inline comment found:**

> //return rtrim($plaintext, "\\0");

***

### decrypt

**Signature:** `decrypt($str)`

**What it likely does:** Decrypt decrypt. likely involves: crypto.

***

### safeEncrypt

**Signature:** `safeEncrypt($plainText)`

**What it likely does:** Provide safeencrypt. likely involves: ip/network.

***

### safeDecrypt

**Signature:** `safeDecrypt($decrypt)`

**What it likely does:** Provide safedecrypt. likely involves: ip/network.

***

### getPostData

**Signature:** `getPostData()`

**What it likely does:** Get getpostdata. likely involves: ip/network.

***

### cleanData

**Signature:** `cleanData($data)`

**What it likely does:** Provide cleandata.

**Inline comment found:**

> //fix for requests that are wrapped in an extra "" for some stupid reason...looking at you cratejoy!

***

### decodeUnicode

**Signature:** `decodeUnicode($str)`

**What it likely does:** Provide decodeunicode.

***

### isRealObject

**Signature:** `isRealObject($arrOrObject)`

**What it likely does:** Check whether isrealobject. likely involves: http request.

***

### curl

**Signature:** `curl($url,$postdata=false,$headers=false,$type='POST',$debug=false,$settings=false,$debugHeaderFn=false)`

**What it likely does:** Provide curl.

***

### getIndexByKey

**Signature:** `getIndexByKey($list,$search,$key)`

**What it likely does:** Get getindexbykey.

**Inline comment found:**

> //die(json\_encode($params));

***

### isJson

**Signature:** `isJson($string)`

**What it likely does:** Check whether isjson.

***

### generateCallTrace

**Signature:** `generateCallTrace()`

**What it likely does:** Generate generatecalltrace.

***

### getUniqueNumber

**Signature:** `getUniqueNumber($zero=false)`

**What it likely does:** Get getuniquenumber.

**Inline comment found:**

> // replace '#someNum' with '$i)', set the right ordering

***

### getUniqueTime

**Signature:** `getUniqueTime($t=false,$zero=false)`

**What it likely does:** Get getuniquetime. likely involves: database.

***

### mail

**Signature:** `mail($site,$campaign,$ropts,$mopts,$js_opts=false)`

**What it likely does:** Provide mail. likely involves: database.

***

### sendMail

**Signature:** `sendMail($obj)`

**What it likely does:** Send sendmail.

**Inline comment found:**

> //die(var\_dump($mopts));

***

### isValidEmail

**Signature:** `isValidEmail($email)`

**What it likely does:** Check whether isvalidemail. likely involves: email.

**Inline comment found:**

> // $log=array(

***

### sendRawMail

**Signature:** `sendRawMail($obj)`

**What it likely does:** Send sendrawmail. likely involves: ip/network, email.

**Inline comment found:**

> // domain is not valid

***

### getObjectKeys

**Signature:** `getObjectKeys($obj,$strip=false)`

**What it likely does:** Get getobjectkeys. likely involves: ip/network.

***

### getObjectKeys2

**Signature:** `getObjectKeys2($obj)`

**What it likely does:** Get getobjectkeys2.

**Inline comment found:**

> //keep key but clear it out

***

### db

**Signature:** `db($site)`

**What it likely does:** Provide db.

***

### getImg

**Signature:** `getImg($obj,$type=false,$s3=false)`

**What it likely does:** Get getimg.

***

### isadmin

**Signature:** `isadmin($site)`

**What it likely does:** Check whether isadmin.

**Inline comment found:**

> //fallback

***

### getDay

**Signature:** `getDay()`

**What it likely does:** Get getday.

***

### uploadProgress

**Signature:** `uploadProgress()`

**What it likely does:** Upload uploadprogress.

***

### relTime

**Signature:** `relTime($seconds)`

**What it likely does:** Provide reltime.

***

### getFiles

**Signature:** `getFiles($dir,$decend=true)`

**What it likely does:** Get getfiles. likely involves: crypto.

**Inline comment found:**

> //days

***

### getDirs

**Signature:** `getDirs($dir)`

**What it likely does:** Get getdirs.

***

### loadProxyImage

**Signature:** `loadProxyImage($r)`

**What it likely does:** Load loadproxyimage.

***

### get\_headers\_from\_curl\_response

**Signature:** `get_headers_from_curl_response($response)`

**What it likely does:** Get headers from curl response.

***

### getUrlsFromString

**Signature:** `getUrlsFromString($string)`

**What it likely does:** Get geturlsfromstring.

***

### getUrlHeaders

**Signature:** `getUrlHeaders($url)`

**What it likely does:** Get geturlheaders.

**Inline comment found:**

> //\[^,\s()\<>]+(?:\(\[\w\d]+\)|(\[^,\[:punct:]\\s]|/))#', $string, $match);

***

### limitLength

**Signature:** `limitLength($text,$length)`

**What it likely does:** Provide limitlength.

***

### getPageTags

**Signature:** `getPageTags($url,$type)`

**What it likely does:** Get getpagetags.

**Inline comment found:**

> //utf8 safe!

***

### getBase64Image

**Signature:** `getBase64Image($url,$clean=false,$mime=false)`

**What it likely does:** Get image. likely involves: image.

**Inline comment found:**

> // supports line breaks inside <title>

***

### getBase64data

**Signature:** `getBase64data($url)`

**What it likely does:** Get data. likely involves: image.

***

### getFileInfo

**Signature:** `getFileInfo($url ,$pretty=false)`

**What it likely does:** Get getfileinfo.

***

### getPath

**Signature:** `getPath($site)`

**What it likely does:** Get getpath. likely involves: database.

**Inline comment found:**

> // return size in bytes

***

### getDomain

**Signature:** `getDomain($url)`

**What it likely does:** Get getdomain.

**Inline comment found:**

> //development

***

### sort

**Signature:** `sort($list,$opts)`

**What it likely does:** Provide sort.

**Inline comment found:**

> //development

***

### getIcons

**Signature:** `getIcons()`

**What it likely does:** Get geticons.

***

### fixFontCss

**Signature:** `fixFontCss($r)`

**What it likely does:** Provide fixfontcss.

***

### getFont64

**Signature:** `getFont64($ext,$url)`

**What it likely does:** Get getfont64.

***

### get\_timezone\_offset

**Signature:** `get_timezone_offset($remote_tz, $origin_tz = null)`

**What it likely does:** Get timezone offset.

***

### getFileConfs

**Signature:** `getFileConfs($key=false)`

**What it likely does:** Get getfileconfs.

***

### publish

**Signature:** `publish($page=false,$suppress=false,$return=false,$force=false)`

**What it likely does:** Provide publish. likely involves: object storage.

***

### renderFileTemplate

**Signature:** `renderFileTemplate($src,$data,$put)`

**What it likely does:** Render renderfiletemplate.

**Inline comment found:**

> // die(json\_encode($dir));

***

### renderTemplate

**Signature:** `renderTemplate($template,$data)`

**What it likely does:** Render rendertemplate.

***

### deleteDirectory

**Signature:** `deleteDirectory($dir)`

**What it likely does:** Delete deletedirectory.

***

### loadCsvData

**Signature:** `loadCsvData($data,$delimiter=',',$rowfn=false,$json=true)`

**What it likely does:** Load loadcsvdata.

***

### copyDirectory

**Signature:** `copyDirectory($src,$dst)`

**What it likely does:** Provide copydirectory.

***

### getByKey

**Signature:** `getByKey($arr,$key,$value)`

**What it likely does:** Get getbykey. likely involves: ip/network.

***

### clear\_tags\_redactor

**Signature:** `clear_tags_redactor($str)`

**What it likely does:** Provide tags redactor. likely involves: ip/network.

***

### rateLimit

**Signature:** `rateLimit($unique_id,$max_count,$time)`

**What it likely does:** Provide ratelimit. likely involves: database.

**Inline comment found:**

> //$time in seconds

***

### decryptCode

**Signature:** `decryptCode($code)`

**What it likely does:** Decrypt decryptcode.

***

### getPurifier

**Signature:** `getPurifier()`

**What it likely does:** Get getpurifier.

***

### ensureRedactorContent

**Signature:** `ensureRedactorContent($content)`

**What it likely does:** Ensure ensureredactorcontent. likely involves: ip/network.

**Inline comment found:**

> // $config->set('HTML.AllowedElements', array('iframe'));// \<-- IMPORTANT

***

### cleanRedactorContent

**Signature:** `cleanRedactorContent($content)`

**What it likely does:** Provide cleanredactorcontent. likely involves: ip/network.

**Inline comment found:**

> // include\_once(ROOT.'/classes/fixmsword.php');

***

### clear\_tags

**Signature:** `clear_tags($str)`

**What it likely does:** Provide tags. likely involves: ip/network.

**Inline comment found:**

> // include\_once(ROOT.'/classes/fixmsword.php');

***

### isVideo

**Signature:** `isVideo($contenttype)`

**What it likely does:** Check whether isvideo. likely involves: ip/network, zip archive.

**Inline comment found:**

> // return fixMSWord(utf8\_decode($content));

***

### mime\_content\_type

**Signature:** `mime_content_type($filename,$returnext=false,$isValid=false)`

**What it likely does:** Provide content type. likely involves: ip/network, zip archive.

***

### isWebsite

**Signature:** `isWebsite($url)`

**What it likely does:** Check whether iswebsite.

***

### getContentType

**Signature:** `getContentType($url)`

**What it likely does:** Get getcontenttype.

**Inline comment found:**

> //die('content: '.$ct);

***

### upload

**Signature:** `upload($r)`

**What it likely does:** Upload upload. likely involves: resize.

***

### exportCSV

**Signature:** `exportCSV($values,$filename='file.csv',$save=false)`

**What it likely does:** Provide exportcsv.

***

### flatten

**Signature:** `flatten($arr)`

**What it likely does:** Provide flatten.

**Inline comment found:**

> // $str='';

***

### getDistanceBetweenPoints

**Signature:** `getDistanceBetweenPoints($pt1,$pt2)`

**What it likely does:** Get getdistancebetweenpoints.

**Inline comment found:**

> // }

***

### getLatLngDistance

**Signature:** `getLatLngDistance($latitudeFrom, $longitudeFrom, $latitudeTo, $longitudeTo, $earthRadiusUnits = 'm')`

**What it likely does:** Get getlatlngdistance.

***

### getDiff

**Signature:** `getDiff($arr1,$arr2)`

**What it likely does:** Get getdiff. likely involves: json response, push notifications.

***

### exportjson

**Signature:** `exportjson($obj,$filename)`

**What it likely does:** Provide exportjson. likely involves: json response.

***

### diejson

**Signature:** `diejson($obj)`

**What it likely does:** Provide diejson. likely involves: json response.

***
