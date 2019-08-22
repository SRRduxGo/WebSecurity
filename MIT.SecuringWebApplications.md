# Securing Web Applications

> [MIT](https://www.youtube.com/watch?v=WlmKwIe9z1Q&list=PLUl4u3cNGP62K2DjQLRxDNRi0z2IRWnNh&index=8)

> [Lecture Notes](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-858-computer-systems-security-fall-2014/lecture-notes/MIT6_858F14_lec9.pdf)

> Shell Shock [BUG](https://en.wikipedia.org/wiki/Shellshock_(software_bug))

## CROSS SITE SCRIPTING DEFENSE

- XSS filters in the browser
  - _Browser detection of script tag in the query parameters_
- can't prevent persistent XSS
- _http only cookies_
  - _A server can tell Browser that Client side JS should not be able to access the particular cookie_
    - _CSRF is still possible in the case_
- _`Privilege Separation`: Use a separate domain for all the content which is un-trusted_
  - _example: Online Service providers: Use a separate domain for user submitted data_
  - _In this case effects of XSS will be limited to that separate domain_
- _`Content Sanitization`: done by web-frameworks- like `React`_
  - _User submitted context is encoded / sanitized in such a way that it will not be able to escape the `static HTML context` and will be contained from doing something malicious_
- _`Using Less expressive MARKUP Language`: (can be called mark-down language) : compiled to native HTML_
- _`Content Security Policy`: Allows a Server to tell a Web-Browser **`what types of content`** can be loaded in the `page` server is sending back and **`where that content should come from`**_
  - _`content-security-policy` header_
    - _`content-security-policy: default-src 'self' *.mydomain.com`_
    - _load content from self ORIGIN or any subdomain of `mydomain.com`_
    - ***DOES NOT ALLOW INLINE JS / EVERY SCRIPT TAG SHOULD HAVE `SRC` ATTRIBUTE / `eval` is not allowed***

> `X-Content-Type-Options: nosniff` : prevents browser from doing helpful optimizations like massaging the `not well formed` content

<hr/>

> _Interestingly we can use meta tag to set CSP. Then what are `meta` [tags](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta)_
>
>```javascript
><meta http-equiv="Content-Security-Policy" content="script-src 'none';">
>```

## SQL Injection Attacks

- **when we directly mix contents from an un-trusted source with a valid SQL Query string**
- _Solution: ***Rigorously encode data from the un-trusted source***_

## Cookies: Session Management

- `Session ID` used as index in some tables
- Beware of domain and sub-domain relationship which can copy or manipulate `Session ID`s
- One way is to have **Stateless Cookies**, then we have to authenticate every request
  - Solution: _MAC - [message authentication code](https://medium.com/@jonmbake/the-why-and-how-of-mac-http-cookies-7fbb2debe47b)_ `H(k,m)` where `H` hashing function, `k` is the key and `m` is the message
  - _Client and Server will share secret key `K`_
    - _**AWS Example**_
      - _AWS gives each user a `USER ID` and a `SECRET KEY` - `k`_
      - _in a request to AWS server user send a header: **`Authorization:'some-string<..user-id..>:some-string<..signature..>'`**_
      - this **`some-string<..signature..>`** encapsulates details of the request
        - _HTTP-VERB : `hverb`_
        - _CONTENT-MD5 : `cm5`_
        - _CONTENT-TYPE : `ct`_
        - _DATE : `dte`_
        - _RESOURCE-NAME : `rname`_
        - _EXPIRATION-DATE : `expDte`_
        - add all of the strings above to create a message `m` = _`hverb + cm5 + ct + dte + rname + expDte`_
        - pass this message to H-Mac function with the secret key `k`, to get _`signature` = `h(k,m)`_
        - if any of the headers are tempered with by an attacker then `Server` would be able to detect it by the difference of the `signature` generated at the server-side from the request headers
      - **QUESTION: where is this secret key? how an attacker cannot get hold of the same**
        - _Client for the AWS is not a Browser, but another machine and user can send a service request to this machine in the middle where this key `k` lives_
      - _There is no expiration or `log out` concept here_
        - _However a server can `revoke` the `secret` key

## Alternatives

- Using DOM Storage in place of a cookie for session-state
  - **`DOM Storage` is strictly bound to the `SINGLE ORIGIN`**
    - ***Less room for manipulating rules to hack the data out***
- _Using Client-Side Cert's like `X.509`_
  - _JS doesn't have `explicit interface` to access `Certs` therefore less likely to become targets of an attack_
  - _Problems:_  
    - _Cert Revocation is hard_
    - _Usability - install certs for trusted sites_

## [Protocol Vulnerabilities](https://youtu.be/WlmKwIe9z1Q?list=PLUl4u3cNGP62K2DjQLRxDNRi0z2IRWnNh&t=3830)

> waste of time this section

- URL Parsing - http://example.com:80@foo.com - Browser parses this URL and ascertains that `foo.com` to be the host
- `.JAR` - hide JAR inside GIF - GIFAR attack

## Time as Vector of Attacks - Covert Channel Attack

- CSS based sniffing attack
  - attacker creates a website and makes a victim visit it
  - Goal - What all other websites the Victim has visited?
  - Exploit - `Links` change `color` if the website underneath it is already visited
  - attacker can present the list of the links to the victim and `sniff the automatic change in the Link color` if the corresponding site has already been visited
  
- Cache Based Attack
  - Attacker makes the victim visit the page attacker.com
  - At attacker.com, makes few requests to pre-destined sites
  - if these resources get loaded (if from cache) much before some threshold time then  attacker can assume the request URL had been visited before by the victim

- DNS based attacks
  - DNS name resolution data is cached
  - and again attacker can make requests to URL's to check how much time it took them to load and factor out the DNS lookup time to guess if the same DNS lookup was made earlier at the User's machine
  - DNS cache lives with OS not with the browser so even if user doesn't allow cache at browser level, it has no control over DNS cache

- Rendering Attacks
  - Since rendering info can also be cached
  - Used with an iframe -- how quickly one's loses control of an iframe if is loading cross-origin site

> Finito ;)