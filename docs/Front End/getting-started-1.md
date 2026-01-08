---
title: Getting Started
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
To install the front end repo, please ensure you have an account created on Gitlab and access to front end code. Reach out to [juicy@actualize.earth](mailto:juicy@actualize.earth) if you need access to a front end repo.

```text
git clone https://gitlab.com/actualize_earth/app-core.git
cd app-core
npm install
npm link
npm install nodemon -g
npm start
```

<br />

# Enabling Developer Mode

Go to [Actualize Web Interface](https://app.actualize.earth) and create/log in to your profile. Make sure to load it in a mobile view. Click on your profile picture in the top right, then tap on the version number at the bottom 5x to enable developer mode. You will then see a "Restart in Local Mode", click that and then "Use Local Development Code (3333)"

<Image align="center" alt="dev_mode" border={false} width="200px" src="https://s3.amazonaws.com/one-earth/static/enable_dev.png" />

<Image align="center" alt="dev_mode_enabled" border={false} width="200px" src="https://s3.amazonaws.com/one-earth/static/dev_testing.png" />

For Mobile app views, load the site in a mobile view using your browser development tools. I recommend viewing the code and the app view in the same window at the same time, like this.

<Image alt="docs" border={false} src="https://one-earth.s3.amazonaws.com/static/code_editor.png" />

You should see the change happen immediately in your browser.

When adding a new view (`/app/views/[new_view].view`), you need to register the view in the conf.json file.

_note_ Error catching has not been perfected yet, so its possible that you may run into a situation where things arent changing or loading. First fix is reload the page and try again. If its still broken, there is probably an issue with templates or the logic. check your developers console to look for messages that may help.

Templating is done with EJS, you can learn more about how EJS works [here](https://ejs.co/#docs).

# Playing with some code

Go to the code editor of your choice.

The relevant files for just getting started are in the

```
/app/views (directory)
```

# Using Branches of the Code

Branches allow different versions of code to co-exists and be merge-able with each other.

Creating a branch

```
git checkout -b [branch_name] 
```

Publishing changes to code repository

```
git add .
git commit -am "[Message to commit, please be as descriptive as possible]"
git push origin [branch_name]
```

Wanting to Publish your own branch and have it accessible in the App? Talk to Juicy about configuring your credentials to give permission for this.

# Basic Understanding of the .view file

There are 5 main sections that can be used in a view, defined by the following tags

```html
<dependencies></dependencies> //Used to dynamically load / Lazy load modules necessary for the view
<route></route> //This registers the view that makes it accessible via link="/[view_name]"
<script></script> //Define all javascript functions here
<templates></templates> //Define all templates used in the view (PLEASE PREFIX ANY TEMPLATES USED WITH THE VIEW NAME, eg @@@[view_name]_home@@@)
<style></style> //Define any CSS (PLEASE PREFIX ANY CSS CLASSES USED WITH THE VIEW NAME, eg .[view_name]_classname)
```

# Notable aspects of templating / linking

HTML Attributes

`action="[event type]:[Function in Context]"` - Used to link an event to a function within the context. EG action="click:alert" will call a this.alert=function(){} within the context of the view if it exists.

`link="/route/to/go"` - Used to navigate to another page / view within the app. Routes are made first by the view name, eg event.view gets a route to `/event/[event.id]`. Additional variables can be passed/used by the view in however they want.

Views are assigned a route when `<route></route>` tag is present in the the `[file_name].view` file. The view can then be accessed by using `link="/[file_name]"`. Or if it is within javascript, you can use `app.history.go('/file_name')`.

Within a view, there are a few command methods to be aware of. The first two (renderOptions and showOptions) are _required_

```javascript
this.renderOptions={
	template:'add', //template to initially render for this view.  A view can have multiple templates in it, but this is the first to be rendered
	uid:'add', // a unique ID for this template - it will auto-assign if it is not set
	append:true, // how to add it to the DOM.  Could be append true/false.  Can also use the replace option
	replace: [Dom Element] - if a dom element is passed to replace it will replace the dom element with the newly rendered content
	context:this, // explicitly passing the context (this) connects all the Javascript in the current view to any bindings that happen in the template - generally this will be "this".  
}
this.showOptions={
	display:'page', //['page' (mobile view, page will show sliding from right to left and will animate the page underneath at 1/2 speed.  This also works with scrollable elements to swipe back to previous page),'page_overlay' (on mobile shows from the bottom up and covers the screen),'page_overlay_web' (shows as a modal view in web, animating up from the bottom)]
	renderOnViewReady:1 //if using a this.getPageData(){} to fetch data on page load, this will re-render the view with the data that was fetched.  This will be depreciated at some point when "reactivity" is added
}
//this is a function to fetch data on the view load from the server.  As it is asyncronous, eg initiating action and some time later return with a result, a callback function is passed (cb) and is called when the necessary data fetching / processing is done.
this.getPageData(cb,reload){
	if(!reload&&self.store.resp&&!self.store.resp.error) return cb();//already have data!
	modules.api({
        url:app.sapiurl+'/module/profile/'+this.options.data.id+'/load',
        data:{},
        timeout:5000,
        callback:function(resp){
        	self.store.resp=resp;
        	cb();
        }
    });
}
//the store is unique to a view and is used to "cache" any relevant data about a view.  This is a persistent store for the view, so when reloading or re-rendering, the data will be saved / used.  Useful for page load data or menu states.
self.store={}
//hooks are event driven listeners based on the life cycle of a view.
this.onInit=function(){} //when the view first loads, do any pre-processing or setting up of the view. First logic to run before anything in the view is processed or rendered.
this.onRender=function(){} // when the view renders.  This happens on ANY render of the main view defined in this.renderOptions.  If loading data dynamically, you might need to look at the data stored to determine the state for which bindings need to happen if you are calling functions in javascript to bind actions.
this.onStart=function(){} // when the view starts / ie becomes visible and active front view
this.onStop=function(){} // when the view goes into the background
this.onResume=function(){} // when a view goes from inactive to active (eg a back navigation will trigger onResume to the view that you are navigating back to).  This also triggers when an app is put to sleep on the phone and then the app comes back into focus.
this.onDestroy=function(){} //any logic needed to clean-up the view before being destroyed
```

By default when using link="" or app.history.go(), the view will render to the default view container based on this logic. It is also possible to directly register a view. This is useful if there are more options that you need to pass than just showing a view. You do this by calling:

```javascript
phi.registerView('[view_name]',{
	renderTo:$('#wrapper')[0],//where to render to
	data:{//any data you want to pass as options to this view
		id:'UBY0U7L3GFPX'
	}
});
```

`bind=""` - A template shortcut to load other componets into a view

```html
bind="loadError:<%=_util.formatOptions({
	ele:'$',
	error:'Not Found',
	onBack:function(){
		this.goBack()
	},
	onRetry:function(){
		var self=this;
		this.getPageData(function(){
            self.refresh();
        });
	}
})%>"
```

References a module to load in the template and the options to pass. As objects / DOM elements cannot be passed, there is a hydration step that happens, so passing an element is possible by a shorthand method. EG ele:'$' will pass a jQuery element of the element the component is rendered within. You can also chain these, like $:.className, where it will first get the parent and then so a search in the DOM for an element with a class "className".

# Templates

Each View can have multiple templates that are used. They are defined in the `<templates></templates>` tags, which looks like this

```html
<templates>
@@@[app_view]_home@@@
<div class="sfit" style="background:black;color:white">
	<div>HOME SCREEN</div>
	<div class="extra_area"></div>
</div>
@@@[app_view]_extra_template@@@
<div>Extra Content</div>
</templates>
```

Rendering additional templates into a view, used in functions in `<script></script>` tags

```javascript
phi.render([jquery or dom element],{ //eg self.ele.find('.extra_area')
	template:'[template name]', //eg '[app_view]_extra_template',
	data:{}, //any data you might want to pass into the template
	contextElement:'string|false', //IMPORTANT this will store a reference of the element in context.  If not provided, it can overwrite a previous context element with the default 'ele'
	context:this // this ensures the current context is use
});
```

# History / Routing (Basic)

Each page handles its routing internally through two functions.

```javascript
this.setRoute=function(){
	app.history.set({
		intent:self.getRoute()
	})
}
this.getRoute=function(){
	return '/blank'
}
```

# Useful App Variables Available

On the app boot, we connect with an Actualize Server to provide dynamic variables based on the environment. This includes things like server endpoint bases. Here is a list of some of the relevant ones. Below are listed for production environment, development environment backends follow a similar convention, however all subdomains are adapted to point to the development server configured based on developer. EG [https://api.actualize.earth](https://api.actualize.earth) points to production API whereas [https://api-juicy.actualize.earth](https://api-juicy.actualize.earth) points to the API on my development server.

```json
{
	appid: "atj4yr10pt57d3",
	sapiurl: "https://api.actualize.earth/core",
	coreapi: "https://api.actualize.earth/core",
	webhookurl: "https://api.actualize.earth/webhook",
	adminurl: "https://api.actualize.earth/admin",
	baseapi: "https://api.actualize.earth",
	chatterapi: "https://api.actualize.earth:3334",
	apiurl2: "https://api.actualize.earth:3333",
	imgurl: "https://img.actualize.earth",
	pageurl: "https://pages.actualize.earth",
	oauthurl: "https://api.actualize.earth/oauth2",
	uploadurl: "https://api.actualize.earth",
	codeurl: "https://code.actualize.earth",
	iconurl: "https://icons.actualize.earth",
	renderurl: "https://render.actualize.earth",
	siteurl: "https://app.actualize.earth",
	linkurl: "https://link.actualize.earth",
	mapurl: "https://api.actualize.earth/map",
	domain: "https://actualize.earth",
	playerurl: "https://player.actualize.earth"
}
```

# Icons

We use fontello's open source icon library builder to customize the icons we use in the project. The current icon set can be found [here](https://icons.actualize.earth).

They can be used like this

```html
<i class="icon-check-1"></i>
```

# Useful global CSS classes

There are a number of CSS classes that can be useful when creating views. Some of these are listed below

```css
.sfit => using position absolute TLRB:0, essentially fits a container to the container its rendered to
.coverimg => Used on <div style="background-image:url([image URL])"> to COVER the area of the element with the image defined
.fitimg => Used on <div style="background-image:url([image URL])"> to FIT the area of the element with the image defined
.[s,m,l]-corner-[tl,tr,bl,br,all] to provide cornering to all or specific corners of elements. s=small, m=medium, l=large
.c-corner-[tl,tr,br,all] to provide "circle" cornering. Use this on buttons when possible.
.mobileheader //used to set appropriate height/z-index of a header based on the phone's header height
.mobilepage //used to set appropriate z-index based on header height so content can scroll behind the header
.mobilespacer //used to set appropriate padding within a scroller so content starts at bottom of scrolling area so content can scroll behind the header

```

Themeing CSS

```css
.themebg => Background color based on theme (EG dark or light mode)
.themetext => Font color based on the theme (Eg dark or light mode)
.button0 => Plain button with themes border color
.button1 => Colored button with themes border color
.button2 => Alt Colored button with themes boarder color
.frostedbg => creates a semi-transparent but "frosted" background.  Useful when content is scrollign behind something to create a modern look
```

Inset Spacing (accounting for phone status/footer bars)

```css
/*
var [top] = phone.insets.data.top;
var [bottom] = phone.insets.data.bottom;
var [top_header] = (phone.insets.data.top+phone.insets.headerHeight);
var [top_header_20] = phone.insets.data.top+phone.insets.headerHeight+20
var [bottom_nav]=(phone.insets.data.bottom+phone.insets.footerHeight;
*/

.mobiletop{
  top:[top_header]px !important;
}
.mobiletop2{
  top:[top_header_20]px !important;
}
.mobileheader{
  padding-top:[top]px !important;
}
.mobilestatusbartop{
  top:[top]px !important;
}
.mobilespacer{
  height:[top_header]px !important;
}
.mobilescrollcontent{
  padding-top:[top_header]px !important;
}
.mobilepageresponsive{
  top:[top_header]px !important;
}
.infinitescroll_sticky_container{
  top:[top_header]px !important;
}
.infinitescroll_sticky_container_top{
  padding-top:[top]px !important;
}
.mobilestatusbar{
  height:[top]px !important;
}
.mobilefooter{
  padding-bottom:[bottom]px !important;
}
.keyboardpage{
  bottom:[bottom]px !important;
}
.keyboardpagenav,.mobilebottom{
   bottom:[bottom_nav]px !important;
}
```

<br />
