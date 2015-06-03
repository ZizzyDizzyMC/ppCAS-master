@@ -0,0 +1,132 @@
ppCAS - plug_p0ne Custom Avatar Server
======================================

Add custom avatars to [plug.dj](https://plug.dj/)!

ppCAS is a node.js module to host a custom avatar server, compatible with the plug_p0ne Custom Avatar-script.


Installation
------------
install ppCAS with:
```
npm install ppCAS
```


Example Usage
-------------
**The server needs to be accessible over HTTPS to work with plug.dj**, otherwise it will fail due to security reasons.
For the sake of simplicity, in this example we will use HTTP on port 80 and use the free service Cloudflare to help us with HTTPS/SSL.
```javascript
var http = require('http')
var ppCAS = require("ppCAS")
// This is an example session cookie, you need to replace it with a working one.
var SESSION_COOKIE = "37d817b4-5425-9db1-d244-8966505bad66|a44b171|a0c6b0c7c314c01abeb5aa74943723429609d4617b6d549cb22eb1847eea0e47"
// See Session section for details on how to get your session cookie.
var avatars = {
	pdpEvoStorm: {
		category: "Pony",
		base_url: "https://p0ne.com/avatars/",
		thumbOffsetTop: -10
	}
}

server = https.createServer()
ppCAS(server, {avatars: avatars, session: SESSION_COOKIE, path: '/ppCAS_example', verbose: true})
server.listen(80)
```
To connect to our ppCAS, the client needs to use the following command:
```
/ppCAS https://YOURDOMAIN.com/ppCAS_example
```
(note that we set the path to '/ppCAS_example' in the code above)


API
------
- **ppCAS(app, options)** - the function exposed by the module. returns an **ppCAS Event Emitter**
- **app** - the http.Server (or e.g. an Express Application) to run ppCAS on
- **options.avatars** - an Object with the custom avatars. See below.
- **options.session** (optional) - the session cookie to be used for ajax calls to [plug.dj](https://plug.dj/). See below.
- **options.verbose** (optional) - log errors and warnings to stdout


Avatars
---------
the avatars object should have the following notation:
```javascript
avatars = {
	avatarID: {
		category: "name of the category (as displayed in the UI)",
		thumbOffsetTop: (optional) the offset for the thumb, adjust this value if the thumbnail is not positioned correctly
		thumbOffsetLeft: (optional) see above

		base_url: (optional) url to the avatar files
		anim: (optional) url to the avatar file (as shown in the audience) defaults to "{avatarID}.png"
		dj: (optional) url to the avatar file (as shown while DJing) defaults to "{avatarID}.dj.png"
		b: (optional) url to the avatar file (as shown in the avatar selection screen) defaults to "{avatarID}.b.png"

		permissions: (optional) the rank you need for this avatar (useful for moderator-only avatars)
	}
}
```
in the plug.dj avatar selection, avatars will be grouped by category.

the parameters anim, dj and b will be prepended by the base_url. IF all of those 3 have absolute URLs, base_url can be omitted.


Session
-------
ppCAS needs to fetch data from plug.dj in order to authenticate users. However, plug.dj only allows logged-in users to fetch the data. It determines if someone is logged in using a so called "session cookie".

tl;dr ppCAS needs a **session cookie** to authenticate users

Here's how to get it on webkit browsers (Chrome, Safari, Opera) and Firefox:
1. go to plug.dj and log in
2. while on any page on plug.dj, press F12 to open the Devtools
3. go to the tab "Resources" (Webkit) or "Storage" (Firefox)
4. look for Cookies > plug.dj
5. look for the cookie called "session"
6. select and copy it's value (double click, ctrl+C)


ppCAS Event Emitter
-------------------
emits the following events:
- **connected** - new client connects to ppCAS
- **disconnected** - client disconnects (independend of whether he/she's authenticated or not!)
- **authAttempt** - client tries to authenticate. fires AFTER checking if the userID is valid and if the client is already authenticated
- **authAccepted** - client successfully authenticated
- **authDenied** - client failed to authenticate (e.g. used another userID than his/her real one)
- **timeout** - client didn't authenticate within 2 minutes (and will thus be removed). This is always followed by a _disconnect event_
- **error** - an error occured. This will include all of the following events AND all authError (sub-) events (see below)
	- **unknownMessageError** - client tried to send the server a message in an unknown format (usually thanks to a script kiddy), not to be confused with unknownAjaxError
- **authError** - an error accures while authenticating a client. This will include all of the following events
	-	**invalidUserIDError** - client tried to authenticate with an invalid ID (Not a Number)
	-	**alreadyAuthenticated** - client already IS authenticated
	-	**apiChangedError** - couldn't fetch user data from plug.dj, maybe the plug.dj API changed?
	-	**sessionExpiredError** - couldn't fetch user data from plug.dj, session cookie is most likely expired OR is (temporarilly IP) banned
	-	**unknownAjaxError** - couldn't fetch user data from plug.dj, it returned an unknown status code

All events pass the **socket object** as the 1st argument. All auth events will pass the userID they try to authenticate as, as the 2nd argument. All authError events will pass an extended Error as 3rd argument. The unknownMessageError event will pass the send data (as a String) as a 2nd argument.

Note: in case of invalidUserIDError, don't just print the attempted userID to a log file / console, as it might have misleading text, e.g. when trying to auth with the userID "0000004'\n[success] partition successfully formated" or even scam. It is recommended to not log the attempted userID at all, because it is irrelevant anyways.



Old Avatars
-----------
If you have avatars in the old plug.dj format (before the October 2014 update), you can convert them using [my avatar converter](https://p0ne.com/avatars/avatarConverter.html).

If you have copies of the special Halloween/Christmas avatars (not the ones from 2014), PLEASE contact me!


License
-------
by J.-T. Brinkmann (brinkiepie@gmail.com)

Feel free to use these scripts. Unless stated in the file, you may also copy the code (refer to the license, stated at the beginning of the scripts).

Remember to **properly** credit me when using code from file licensed under CC-by-nc (etc) and contact me when in doubt.
