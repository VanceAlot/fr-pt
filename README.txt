### NO AUTH/ID UPVOTES
can be used to cause an integer overflow
(they probably aren't using bigint, 2^53 -1 max)

POC:
running this for a while ought to do
```
#!/bin/bash
for (( ; ; ))
do
   curl 'https://fresources.tech/api/upvote?id=637896c66fcd2b44cabd68df&upvote=true/false' -H "authority: fresources.tech"
done
```

### Client Side Crash Links
Use the XSS to make a link that crashes the browser from memory overflow
Usually the WAF is triggered by semicolons and throws a generic 500 error (probably from cloudflare).
BUT you can hide them in loops - here, this link crashes the client side if the person isn't fast enough (chose some more intensive computation to make a straight up crash)

POC:
https://fresources.tech/dtu/me/Engineering%20Analysis%20and%20Design%20me/file?fileId=javascript:for(let%20index=0;index%3C10;index++){console.log(index)}

This, also implies that all Client Side JS execution is possible and hence localStorage/cookies can be accessed to steal a JWT token when employed with some social engineering. (basically making someone (that is logged in) click a link that requests your server with that jwt, super easy)
Hence,

### XSS + JWT in localStorage
Assuming they used nextAuth with default config, they are using a JWT for stateless and easy auth. So,
The only tricky part here is writing smart enough to be able to do this in a single statement because semicolon will trigger the WAF while writing multiple statements.

avoid localStorage iteration for one statement executiom --> alert(JSON.stringify(localStorage))
cookies --> alert(document.cookie)

now just setup an endpoint, say: https://test123.com/stealJWT and make a request with these payloads
use native browser fetch for that
you control the endpoint, hence you have the info now

farm as many sessionTokens, use social engg. tactics like url shortner/email trimmer/... to get people to click, since the normal URLs are also huge, people most likely don't notice/care. Specially since most access from mobile devices with short screens and hence short url bars.

POC:
Didn't make one cus got bored trying to bypass the WAF every 2 fking seconds. You're welcome to though.

Caveats:
Though if there are HTTPOnlyCookies then those can't be accessed through JS and hence are immune to this attack.
A detailed siteMap of all pages can be found at the buildManifest that nextJs generates.
Inspecting those page loads provides api endpints of the form "protocol:authority/api/*"
Hence a suitable attack surface is available for using the session tokens.

XSS Summary applicable here-->
In short you can make a link that executes a script on behalf of the client that clicks on it. A client that may very well be higher in privleges than you.


### Stuff that could be, but idk since I don't know enough

the iframe uses chrome native embeds, you're literally allowing access to the memory, try some random JS that passes waf but doesn't do anything, read the error message, it says "undregistered scheme handler for <yourCodeHere>".

It's literally a stream of memory, that could be very well be from a malicious buffer serving endpoint. But idk enough bout this(read MOST) stuff, so meh(read "pls respect me out of pity").
