Title: Bypass Same Origin Policy on IE11 with a Null Origin
Date: 2-21-2016
Tags: WebApp Security, Same Origin Policy, Bypass, IE11
Slug: bypass-same-origin-policy-ie11-null-origin
Author: Caleb
Summary: It is possible to bypass the Same Origin policy on IE11 (all tested versions so far.) This can be done by having a site that used the Origin NULL. NULL origins are used by locally loaded HTML.

The Background
--------------
Most browsers implement a Same-Origin policy (SOP). This means that resources from one origin can no be shared with a different origin (aka url, or domain). All major browsers do this. Chrome, Firefox, Edge, IE, Safari, etc. They all also have a way to bypass this restriction, a Cross-Origin Resource Sharing request (CORS). Basically, one website sends it's origin (domain) to another, and asks if it has permission to send requests cross-origin. This is done using an OPTIONS request. If the response has adequate headers for that domain, the browser will allow that domain to send near about any request to the other domain. 

The Fun Part
------------
I have found an issue with the above scenario while using IE11 (all versions from at least 11.0.9600.17801 through the current version, 11.0.9600.18059.) If a website has a NULL origin, IE11 does not first send the CORS Preflight OPTIONS request. It will instead just send the Cross Origin request (assuming the user has pressed "allow blocked content")

![IE11 asks the user to "Allow Blocked Content"](/images/bypass-same-origin-policy-ie11-null-origin01-Allow-Blocked-Content.png)

![IE11 Sends the DELETE request using Application/JSON without an OPTIONS pre-flight](/images/bypass-same-origin-policy-ie11-null-origin01-no-OPTIONS.png)

In a given scenario, a website wants to send a request with `content-type: application/json.`

![A "dangerous" request, it is HTTP method DELETE, and uses a content-type of application/json](/images/bypass-same-origin-policy-ie11-null-origin01-Dangerous-Request.png)

In Chrome, the first site will first ask permission (as Chrome enforces), by sending an OPTIONS request. The receiving site denies permissions, and Chrome will not send the Cross-Origin request. The same thing happens in IE11, unless the site has a NULL origin. In Chrome, a NULL origin site still has to send a preflight request and obtain permission. In IE, if the Origin is NULL, IE will disregard the CORS policies, and send the full request regardless. 

![Chrome sends the required Pre-Flight OPTIONS request, unlike the IE request shown just below that that sent the DELETE right away.](/images/bypass-same-origin-policy-ie11-null-origin01-Pre-Flight-Sent.png)


The Specification
-----------------
According to W3C, [Cross-Origin Resource Sharing recommendation](https://www.w3.org/TR/2014/REC-cors-20140116/#resource-preflight-requests "CORS"), IF the Origin header is set, the Pre-Flight must be sent for these requests, however when the Origin is null, IE11 does not set the header at all. Per the recommendation: `If the Origin header is not present terminate this set of steps. The request is outside the scope of this specification.`
So IE is not outside of the specification, in that the header is not being set (although the Origin is null, no Origin header is set.) 

So we go to [RFC 6454](https://www.ietf.org/rfc/rfc6454.txt "Origin") to see when an origin header should be set. Per section 7.3, User Agent Requirements: `The user agent MAY include an Origin field in any HTTP request.`

So IE11 is not technically out of specification with RFC6464. The spec is broad, in that an Origin header is never required. All other modern browsers I have tested seem to set a NULL header (also within specification.) So why does IE not do so?

Now the question, how do we get a NULL origin? This is not really simple, in fact if an attacker wanted to abuse this flaw, they would have to get you to execute HTML from your local disk. If IE opens a web page from a local file, that "origin" is set to NULL. In this case, any Cross Origin requests would be sent without asking the other site for permission.

Thoughts
--------
Now this attack vector is pretty small, and basically falls under the "Don't run anything on your machine that you do not trust" adage.
I have reported this to Microsoft, and given the small attack vector, they do not feel it is an issue. Given that if an attacker can convince you to run something on your system, they will more likely have you run malicious code, rather than HTML to perform an XSS or CSRF attack. 

![Microsoft says this is a non-issue](/images/bypass-same-origin-policy-ie11-null-origin01-Non-issue-From-MS.png)

Although I personally agree the attack vector is very small, if any flaw were found in the future to allow you to set your origin to NULL, this could be abused (if you could set your origin in general, CORS would be bypassed anyway). 

If you know of any way to set your origin to NULL in IE, please let me know. I would love to test this further.



Versions Tested
---------------
Internet Explorer 11 Versions:
*	11.0.9600.17416, 
*	11.0.9600.17801, 
*	11.0.9600.18059, 
*	11.0.10240.16590