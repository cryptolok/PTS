# PTS

[Private Token Sharing](https://private-token-sharing.net/)

![](https://raw.githubusercontent.com/cryptolok/PTS/master/logo.png)

PTS is a web service for secure and private password sharing with one-time tokens and end-to-end encryption

Properties:
* security
* portability
* compatibility
* minimalism
* resilience
* versatility
* deniability

Dependencies:
* **Apache/Nginx with PHP and modules** - main server engine
* **JavaScript** - main client engine, almost any browser is supported, including Tor

Limitations:
* no proper RAM cleaning
* static password policy
* impossibility to store unpredictable constants
* susceptible to MITM (rogue certificate or domain, first HTTP connection)
* application and server security and trust can't be 100%

## How it works

The solution allows to share a random password (token) securely and privately between two parties in their browsers.

![](https://raw.githubusercontent.com/cryptolok/PTS/master/schema.png)

1. Alice makes a secure HTTPS connection to the Server
	* Alice generates random 12 bytes (~100 entropy bits), then Base64 encodes them, producing 16 ASCII characters and adding "/0" to the end (token)
	* Alice generates another random 32 bytes (~250 entropy bits), then Base64 encodes them, producing 43 ASCII characters (password)
	* Alice encrypts the token (12 bytes) with the password (32 bytes) and sends it to the Server for storage during one week
	* Server returns the URL (16 characters) by which the encrypted token can be accessed (id)
	* Alice adds the password to the URL using hashtag (#)
2. Alice sends in clear text the URL to Bob
3. Bob makes a secure HTTPS connection to the Server
	* Bob sends the id of the encrypted token to the Server
	* If the Server finds it and if it has been less than one week, it returns it to Bob and deletes it from storage or past one week
	* Bob uses the password after hashtag to decrypt the token
4. If Bob succeeds at the decryption, Bob confirms the token reception to Alice in clear text
5. Alice and Bob now share a token that they can use for an encrypted communication

If any step if failed, then the schema is started over.

Everything is transparent and secure for clients and server doesn't operate or store data in clear. The password after hashtag in the URL isn't sent to the server either.

It's like a quantum public key exchange cryptography with one-time accessible tokens.

It uses [PrivateBin](https://privatebin.info/) for storage and cryptographic operations as main part.

### HowTo

If you just want to use it, access my [online](https://private-token-sharing.net/) web site.

Example of usage :
Lets say, you want to share a file, so you decide to encrypt it, but you want to share a common password first
1. By visiting the site, it will automatically generate a token with the associated URL
![](https://raw.githubusercontent.com/cryptolok/PTS/master/usage1.png)
Now, don't use the token directly or save it for a later usage and send the generated link to another party, don't send any encrypted data yet (but you can use the token to ecnrypt it)
2. Another party will access the link and see the token
![](https://raw.githubusercontent.com/cryptolok/PTS/master/usage2.png)
Once again, don't use the token before the other party confirms its successful reception, afterwards, the token can be used for both parties (encrypted file can be send and thus safely decrypted)
3. The token will be deleted subsequently and if another party will see such message, it means that the token has to be regenerated and can't be used
![](https://raw.githubusercontent.com/cryptolok/PTS/master/usage3.png)

If you want to install the service yourself :
1. Install Apache/Nginx with PHP
1. Install [PrivateBin](https://github.com/PrivateBin/PrivateBin/blob/master/INSTALL.md#installation) with associated PHP modules
3. Configure [SSL](https://mozilla.github.io/server-side-tls/ssl-config-generator/), [certificate](https://letsencrypt.org/getting-started/) and [Security Headers](https://geekflare.com/http-header-implementation/)
4. Rename /var/www/html/index.php to privatebin.php
5. Put index.html and logo.png to /var/www/html/

#### Analysis

As you can see, my solution uses cloud 2E2 cryptography to achieve zero-knowledge and deniability.

The data are encrypted with AES-256-GCM and the pseudo-random generation is based on [Fortuna](https://en.wikipedia.org/wiki/Fortuna_(PRNG)) including entropy check.

Adding "/0" to the end of the token is preferable in order to make sure that there is at least one number and a special character.

JavaScript libraries are verified for integrity and supported by almost any browser.

My server has custom security configuration regarding system, network, SSL/TLS and application.


Now, let's see some attack scenarios.

The URL is transmitted in clear, it can be intercepted. In case the attacker access the token before Bob, Bob won't be able to see the token, so it makes no sense. It is interesting, however, if the attacker intercepts the request, makes another one, and replace the original with its own, thus allowing to intercept and re-encrypt any further communication. Parties could, nevertheless, verify the integrity of their tokens by exchanging hashes, although, it can be more complicated for an attacker if done by another communication channel (email, SMS, messenger, etc.), the attack is still plausible.

Unfortunately, such interception can be done on multiple levels, an attacker can redirect domain name or have a backdoor in one of your certificate authorities, as well as intercept the very first HTTP connection to the server, thus bypassing HSTS. It can be your neighbor or your government.

That said, the most practical way to break the security is MITM.

I wouldn't exclude time analysis attack either (the time between site visiting and the server request), as well as data storage in RAM.

Replay attacks and passive interception are mitigated though.

##### Notes

Technology, especially cryptography coupled with logic and creativity can be used as a decent meaning of achieving privacy and security.

Privacy isn't necesserily sacrificed for security and vice verca.

> "A career is born in public - talent in privacy."

Marilyn Monroe
