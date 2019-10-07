# [SSL and HTTPS](https://youtu.be/q1OF_0ICt9A?list=PLUl4u3cNGP62K2DjQLRxDNRi0z2IRWnNh)

> ## Symmetric enc / dec
>
> `P = Data`, `E = Encryption function` with some key `k`  
> `E(P) = C`, `D = Decryption function` with the same key `k`  
> `D(C) = P`
>
> ## Asymmetric enc / dec
>
> `P = Data`, `E = Encryption function` with some public key `pk`  
> `E(P) = C`, `D = Decryption function` with corresponding `secret key` `sk`  
> `D(C) = P`  
> In practice you sign or verify message  
> _`Sign(m,sk) = S` `Verify(m,S,pk) = OK:YES or NO?`_
>
> Another Example  
> `A` knows `B`'s `public key - pk`  
> `A` will generate a Random secret key `S` and will send it too `B` using `pk`  
> Once `B` has the secret key `A` and `B` can communicate using the secret key `S` > \_\_

## Discover PK

- Certificate Authority (CA) : Maintains a table of Principle against Public Key
- CA signs the Message `{name:Public Key}` using CA's `secret key` - this is called the `Certificate`
  - _`Server and the Browser` send their respective `certificates` to each other to gain `trust`_
  - These certificates include an expiration time
  - Risks - wrong certificate handed over, Secret key getting leaked
    - _To mitigate the risk - CRL - Certificate Revoked List - Browsers are supposed to download the CRL's_
    - _OCSP (Online Certificate Status Protocol) protocol) - Contacting a dedicated server / service to ask whether the recieved Certificate is valid or not (not used very widely)_
    - _Browsers, `Chrome`, have a baked in `Certificate to be Revoked` list_

### How to integrate the above Cryptography concepts inside Web Browsers

#### Three things to protect in a network

- Data on Network - SSL / TLS
- Code / Data in the browser

  - Issue here is we have to protect the Code / Data coming from an Encrypted connection from Data / Code coming from Non-Encrypted connection
  - Threat Model is - anything un-encrypted can be tempered with by a network attacker which in-turn can temper with Code / Data received over encrypted connection
    - To fix this - new URL scheme is introduced `HTTPS://`
      - Origin is different compared to `HTTP://` - SAME ORIGIN POLICY gets enforced
    - `HostName` in the URL has to be in the certificate
    - Cookies with `Secure` flag are sent to `https://` requests only
    - _BUT if a cookie doesn't have the `Secure` flag then it can be sent to both `http://` and `https://` requests_

- UI
  - _Lock Icon - Indicative of Secure site - Depends on which browser ypu are using and certificate staleness_

#### Code / Data in the browser

- Dns - how important it is?
  - _Might direct user to a site which has stolen an old certificate_
  - _https and tls assumes that DNS can be compromised_
- _if a user accepts a tampered certificate , browser will treat it as a valid certificate going forward, for that particular site and that site will have access to all the respective cookies_
- _Mixed embedding - Suppose you have an `https://` site but insecure embedding like a script tag in a page_

  - _for example - `<script src="http://jquery.com"/>`_
  - _if the target URL is compromised_
  - _And such requests is sent without any encryption or any authentication over the network_
  - _And the compromised `JS`code will run as a part of this page and will have access to cookies_
  - _hence in a secure site always embed secure sources_
  - _or use origin relative urls - `src= "//jquery.com"`_
  - _OR include a hash in the tag of the content you are going to get from the given server - publishing server has to publish these hash values, in this model there is no need to have `https://` url_

- Another scenario - if dev forgets to set the `secure` flag in the `cookie` on a `https:` connection
  - so an attacker can fake/intercept a request requesting for `http:` connection and get the `http:` cookies
  - on `http://` there is no way to authenticate the HOST
  - for example if one visits a site which have embedded `http:` images path for the target site then with all such requests all the insecure cookies of that site will be sent
  - This also has another problem that is - for the secured server there is no way to know if the insecure cookie was set on `http://` or `https://` connection
  - _could lead to session fixation attack_

#### in Forced HTTPS:// or HSTS (http strict transport protocol)

- Any certificate errors are fatal
- it redirects all `http://` to `https://` connection
- _`Prohibits insecure embedding`_
- _SET forced HTTPS bit_
  - _Visiting for the first time a forced https site can be compromised because the bit is not set yet_
