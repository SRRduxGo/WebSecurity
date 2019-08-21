# Web Security Model

> [MIT](https://youtu.be/eRJ_r8WF1Y0?list=PLUl4u3cNGP62K2DjQLRxDNRi0z2IRWnNh)


> [Important Lecture Notes](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-858-computer-systems-security-fall-2014/lecture-notes/MIT6_858F14_lec8.pdf)

## Browser is super complicated

- JS
- DOM model
- XML HTTP requests (AJAX)
- WebSockets
- Multimedia support (`<video tag>`)
- Geo-location
- NaCL

> Threat surface is enormous !!

> Problems arising due to composition between various systems

- example - **Parsing Contexts**

```javascript
<script>
    var x = "THE SOURCE OF THIS STRING COMES FROM AN UN-TRUSTED PARTY CODE"
</script>
```

- ***This un-trusted string can have closing double-quotes / start a new script or end the script***
- _Evolving Specs and implementations with Browser vendor can vary and can open a loopHole for an attacker to manipulate_
- _Resources in a given page might be come from various sources_
- _pages sometimes have frames_
  - _This frame might be showing content from a different context all together_
  - _Relationship between the frame and the main window adds to the dynamic_

## BROWSER's SECURITY MODEL - SAME ORIGIN POLICY

- **Two website's should not be able to tamper with each other unless they want to**
- ***Strategy - Each resource is assigned an ORIGIN***
  - _Javascript can access resources coming from it's own origin_
  - _BUT THERE ARE EXCEPTIONS_
- **ORIGIN = `NETWORK PROTOCOL SCHEME  + HOSTNAME + PORT`**
  - _Two URL's seemingly from the same origin but are not_
    - _http://foo.com/index.html_ ***ORIGIN -> http + foo.com + 80***
    - _https://foo.com/index.html_ ***ORIGIN -> http + foo.com + 443***

### Four Ideas
  
- 1; Each `ORIGIN` has client-side resources
  - _Cookies_
  - _DOM storage_
  - _JS nameSpace_
  - _DOM Tree_ ( is a JS reflection of the corresponding HTML)
  - _Visual Display area_
- 2; Each `frame` gets the `ORIGIN` of its `URL`
- 3; `Scripts` execute with the `Authority of its frame ORIGIN` in which it has been requested as a `resource`
- 4; Passive Content - GETS ZERO AUTHORITY FROM THE BROWSER

#### Frames

- Frames from `different ORIGINS` can communicate with each other via `PostMessage` [interface](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) SUPER COOL 0O/< !
- **Child Frames - Different Origin - cannot launch XML-HTTP request to the parent ORIGIN Host**

#### Idea NO. 4

- MIME types; older IE version use to interpret resource-type by looking at the first 256 bytes of the resource in question
  - _An image from an attacker could have malicious JS in first 256 bytes of code and kaboom !!_

#### Frames / Window Objects

- Gets the `ORIGIN` of the `frame's URL` OR A `suffix` of the Original ORIGIN
  - _through setting document.domain_
  - _Original ORIGIN `x.y.z.com`  => suffix `y.z.com`,`z.com`_
  - _CANNOT DO => `a.y.z.com`, `.com`_
- Two frames can access each other if
  - _`Both set` `document.domain` to the same value_
  - _**Neither** has `set` `document.domain` and there is a value match_
  - It's kind of `NOT.XOR` logic

#### DOM NODES - Get the ORIGIN of the Surrounding frame

#### COOKIES have a Domain + Path

- `*.mit.edu/6.858` => corresponding cookie `Domain = *.mit.edu` and `path = /6.858`
- `(DOMAIN,PATH)` of a cookie can be set by the Server or the Browser
  - _Client side => document.cookie_
  - can also set a `secure` flag from the server side to indicate that its a HTTPS cookie; HTTP content cannot access that cookie
- _When browser makes request to a Server, then all the cookies belonging to that domain and path are sent as a part of that HTTP/s request_
- _[public suffix](publicsuffix.org)_

#### XML HTTP Requests

- Follow same origin policy - unless server has enabled `CORS`
- header - `Access-Control-Allow-Origin`

#### Images

- A `frame` can load images from `any ORIGIN` it desires but `cannot` inspect the bits of the Image

#### CSS

- A `frame` can load `CSS` `from any ORIGIN` but `cannot inspect its contents`

#### JS

- `Cross Origin` fetch for JS is allowed
- `external JS` can `execute` in the context of your page
- But `cannot` look inside the source code for the external JS
- loophole - fun.toString() => gives source-code in string format - that is why minified
- let server `fetch` the JS code if possible and add it to the HTML content / prevents cookies to be sent
- `OBFUSCATION`

#### Plugins

- can be loaded from any `ORIGIN`

#### Browser Generated HTTP requests and COOKIES

- All the relevant cookies for the request (`NETWORK PROTOCOL SCHEME  + HOSTNAME + PORT`) are included (following the `SAME ORIGIN` policy)
- _And if the User is visiting the malicious site and that site makes such a `CROSS ORIGIN request` (to your Bank's host) then all the relevant cookies sitting at user's machine will become part of that request and request would seem valid to the HOST's / bank's server - | if its a XHR request then XHR object should have `XHR.withCredentials=true` to automatically include relevant cookies_
- _`CSRF` - CROSS SITE REQUEST FORGERY_
- _Server Side can prevent this by `Including a Randomly generated CSRF token` which should always be part of any request made by the browser to the Target Host, which means Malicious site has to guess this token value and if `sufficiently RANDOM` then such attacks can be prevented_

#### Network Addresses [MIT](https://youtu.be/eRJ_r8WF1Y0?list=PLUl4u3cNGP62K2DjQLRxDNRi0z2IRWnNh&t=3704)

- _A frame can send HTTP and HTTPS requests to host and port which matches Frame's ORIGIN_
- _**`DNS REBINDING ATTACK`**_ →  Goal: `To run attacker controlled JS with the Authority of Victim.com`.
  - _Attacker wants to bust the same origin policy_
- _[Approach](https://en.wikipedia.org/wiki/DNS_rebinding):_
  - The attacker `registers` a domain (such as `attacker.com`)
  - `delegates` it to a `DNS server` that is `under the attacker's control`
  - The server is `configured` to respond with a `very short time to live (TTL) record` 
    - _preventing the DNS response from being cached_
  - When the victim browses to the malicious domain:
    - the attacker's `DNS` server `first responds` with the `IP address` of a server hosting the `malicious client-side code`
  - The `malicious` client-side code makes `additional accesses` to the `original` domain name (such as `attacker.com`)
  - These are permitted by the `same-origin` policy
  - However, when the victim's browser runs the script `it makes a new DNS request` for the domain
  - the attacker `replies` with a `new IP address`
    - For instance, they could reply with `an internal IP address` or the IP address of a target somewhere else on the Internet

> DNS: Name resolution algorithms are based on Name/ string comparisons rather than on actual IP addresses

- ***`Prevention`***
  - _Modify your client side DNS resolver so that external host-names can never resolve to internal IP Addresses_
  - _DNS pinning: Done by the browser (find more on wiki)_

#### PIXELS

- Pixels don't have an origin
- Each frame gets its own bounding box
- Each frame can draw anywhere on that square
- `Issue` is `a Parent frame` can draw on top of a `Child frame`
  - _leads to `click jacking attack`_
  - _solution:_
    - _Client Side: FrameBustingCode JS solution_
    - _Server side: flag the `X-Frame-Options` header_

> Typo Squatting Attacks;
