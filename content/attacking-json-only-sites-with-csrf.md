Title: Attacking JSON only sites with CSRF
Date: 09-23-2015 13:13
Tags: WebApp Security, CSRF, Red Team
Slug: attacking-json-only-sites-with-csrf
Author: Caleb
Summary: An attacker can still perform CSRF attacks against sites that only respond to JSON formatted text. By manipulating content-type headers, but keeping JSON formatted bodies, a browser will send a JSON-like request without a Preflight request.

The Background
--------------
The web has been built around the idea of a [same origin policy](https://www.w3.org/Security/wiki/Same_Origin_Policy "W3 - Same Origin Policy"), and to get around that, [CORS](http://www.w3.org/TR/cors/ "W3 - Cross-Origin Resource Sharing").

The jist of this is that "odd" requests can not be sent to one domain from another. CORS allows web sites to ignore this, or allow certain domains to send "odd" requests. "Odd" here means anything that is not POST or GET. It also blocks many content types, such as application/xml or application/json. Many API based sites use PUT or DELETE calls in order to make requests. This makes them practically non-CSRF'able. Many sites also use all JSON requests, which should in-theory act the same way. Since an attacker can not force another domain to send application/json to a seperate domain.

The Fun Part
------------
In my experience, many JSON parsers do not quite conform to the JSON RFC. According to the spec, JSON has a content type as follows:

`content-type: application/json`

As such, an attacker can not make their own personal domain send an XHR request with this content type, without the browser flagging it, and sending a CORS preflight reqeust to ask for permission from the other domain. The problem lies in that many JSON parsers do not actually check the Request headers. They are not actually confirming that this was a "JSON" request, so much as it looks like a JSON request. In many cases, an attacker can create an XHR request on their domain to send a body with a JSON formatted text, but includes a content-type header set to text/plain. Because the content type is not "odd", the browser will not flag the request, and will send this XHR request without first sending the CORS preflight request. 

I have tested this in PHP using the default json_decode. It works fine, without additional code, json_decode() does not check the content-type header. As such, any site written in PHP, using all JSON requests, is probably still vulnerable to CSRF attacks. 

Thoughts
--------
Basically OWASP recommends using randomized nonces (a random token) in the body of every request. This token should be long and have good entropy. It should also be a different token for every reqeust, for every session. Further reading can be found at [OWASP - CSRF](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)  "OWASP - CSRF"). It is not sufficient to depend on Same Origin policy. Even with other HTTP Verbs, such as PUT or DELETE. CORS and SOP have been bypassed in the past, and will be again.

As for attackers, before you write off a JSON only site as non-CSRF'able, make sure to try changing up the content-type headers. You may be able to attack this nonetheless.
