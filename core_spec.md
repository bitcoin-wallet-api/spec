
Bitcoin Wallet API Spec
=======================

Motivation
----------

Apps implementing custom schemes on top of Bitcoin blockchain require user to provide bitcoins and signatures in custom transactions. Unified API enables users to keep their bitcoins in a convenient place (personal wallet) while safely allocating desired amounts to custom transactions.

The goal is to avoid 3rd party apps from re-inventing secure key storage and backup. User wallet already keeps the keys, authorizes access to them and implements a backup procedure. Third party app can simply ask user to sign certain transactions and add their bitcoins in them.

Overview
--------

Legend:

- *App*: application that implements some sort of custom transactions.
- *Wallet*: an application or device that is trusted by user and stores his private keys.

Use cases:

1. App is a web page and wallet is another web service (e.g. Blockchain.info, Coinbase).
2. App is a web page and the wallet is a native app.
3. App is a native app and wallet is a native app.
4. App is a web page and wallet is a separate hardware device.
5. Etc.

Structure
---------

Bitcoin apps and wallets exist on a variety of platforms: web sites, native apps, browser extensions, hardware devices. All of these should have means to communicate with each other. 

This specification defines several distinct protocols:

1. **Core Spec** covers common binary data structures and secure protocol of exchanging them without specifying any specific programming interfaces. This protocol is the basis for all other protocols defined below.

2. **[Bitcoin HTTP API](http_api_spec.md)** defines means to communicate core data structures using HTTP requests. This is for wallets hosted on the web like Coinbase and Blockchain.info.

3. **[Bitcoin URL API](url_spec.md)** defines means to communicate core data structures using standard *bitcoin:* URL scheme. This allows simple, although not very transparent, integration between websites and native apps.

4. **[Bitcoin JS API](js_api_spec.md)** defines JavaScript API for web browsers that allows web pages communicating to Bitcoin wallets.

5. **[Bitcoin iOS/OSX extensions](ios_osx_api_spec.md)** defines a standard way to declare and access extensions provided by wallet apps on iOS and OS X (starting with iOS 8 and OS X 10.10).

6. **[Bitcoin Android extentions](android_api_spec.md)** defines a standard way to declare extensions for Android apps.

7. **[Bitcoin Windows extentions](windows_api_spec.md)** defines a standard way to declare extensions for Windows 8 apps.


App developer: what API should I use?
-------------------------------------

If you develop a **web application**, you should consider three APIs: Bitcoin JS API (when available), Bitcoin HTTP API and Bitcoin URL API. Bitcoin JS API provides smoother experience for user, but requires coordination between their web browser and their wallet. Bitcoin HTTP API is also smooth and works similarly to OAuth: user has to authenticate transaction on a separate web page (but it works only with web-based wallets). Bitcoin URL may be used directly without special support from the browser and is used for communicating with native wallet apps. Bitcoin URL API also allows easy fallback to an intermediate wallet that can be used to fund special transactions (in case user's wallet does not support any form of Wallet API).

If you develop a **native application**, e.g. for iOS, consider using native Bitcoin extension API, Bitcoin HTTP API or Bitcoin URL. Same reasoning applies as to web apps. When native extensions are not available, use Bitcoin HTTP for web wallets and URL scheme for native apps. You can use custom URL schemes in URL callbacks for inter-app communication.


Wallet developer: what should I implement?
------------------------------------------

If your wallet is **web-based**, consider implementing Bitcoin HTTP API. Also consider developing a *satellite native app* to ease integration with native apps. 

If your wallet is a **native app**, it is recommended that you implement all three APIs: Bitcoin URL API, native extension for your platform and integrate with browsers of your platforms so they can provide JS API. If the browser on your platform already knows how to speak with native extensions, you do not need to specifically integrate with JS API.

If your wallet is a **separate device**, consider implementing both JS API via browser extension and a native app with a native extension. If the browser on your supported platform already supports JS API by communicating with native extensions, you may simply develop an app with native extension API.

If you are a **web browser** developer, consider these possibilities: 

1. Expose JS API that talks to extensions provided by native wallet apps.
2. Allow connecting to this API by your own in-browser extensions (so instead of redefining JS API from scratch they provide it directly for the browser). This is for wallets that are implemented as in-browser extensions or for in-browser extensions that talk to external hardware or software wallets.
3. Expose the browser itself as a wallet via native extensions for other native apps and proxy requests to the wallets to which it is connected.
4. As a variant of #3, identify web-based bitcoin wallets by <link> meta tag and proxy requests to them via HTTP API.


Core Spec
---------

Bitcoin Wallet API is based on three core protocols:

1. **Pay-to-Transaction**. App sends an incomplete transaction to Wallet and asks to add a certain amount of coins in it. Wallet asks user's permission and adds signed inputs to the transaction.

2. **Inputs Authorization**. This is a two-step process. App requests from Wallet an authorization to spend certain amount of coins. Wallet asks user's permission and sends back to the app a list of unsigned inputs and change outputs. App uses them to compose a complete transaction that is sent back to Wallet for signing. Wallet signs the transaction if it correctly uses all authorized inputs and outputs.

3. **Public Key API**. App may outsource key storage to Wallet because it already implements security and safety measures such as encryption, authorization and backups. App can only access public keys specific to itself and request signatures of arbitrary data using these keys. Wallet never stores its own coins using these keys and only provides storage service to the app. 

4. **Signature API**. Using the public key exposed in the previous API, App may request arbitrary data to be signed by the given key.

5. **Diffie-Hellman API**. Using the public key exposed in the previous API, App may ask Wallet to multiply an arbitrary public key by a private key allocated for this App.


Pay-to-Transaction Specification
--------------------------------

One request: app sends an incomplete transaction and asks wallet to add inputs and outputs and sign inputs. Wallet returns inputs, outputs and signatures.

Signatures may be done with ANYONECANPAY flag.


Inputs Authorization Specification
----------------------------------

Two requests: 

1. Send desired amount and a message, receive inputs and outputs. 
2. Send complete transaction. Wallet signs the inputs if all inputs and outputs are in place and returns signatures.

App is free to supply incomplete transaction and ask wallet to sign with ANYONECANPAY flag.

TODO: specify the timeout for authorization.

TODO: think about explicit revocation of authorization so app may retry.

TODO: require certain number of confirmations on an output from wallet. Rationale: while technically, app can check confirmations of the inputs itself, it will do that *after* getting user's permission to spend funds. If it wants to retry, it would be very confusing for the user to authorize again the same payment. Also, retrying is not a guarantee to receive confirmed unspent outputs again. To simplify things, wallet takes care of it if it can. If app does not care, it specifies number of confirmations equal to 0.


Public Key Specification
------------------------

Public key is linked to identity of the requestor. Different APIs do it differently. 

Public key is indexed by BIP32 non-hardened derivation index. This is to simplify key management for the wallets so they do not need to store any app-specific data at all.


Signature Specification
-----------------------

Standard ECDSA signature. Maybe require it to be deterministic according to RFC6979?


Diffie-Hellman Specification
----------------------------

Wallet should check if the public key is correct and sign.


Proxying
--------

If the implementation of this spec acts as a bridge between the app and the wallet (e.g. a web browser or a native client to a hardware wallet), then it may proxy requests directly to the recipient without asking user's permission too often. 

### JS API proxying by web browser

Browser may ask user to select a wallet of his choice or detect such wallet automatically (using bitcoin: scheme for native apps). 

Browser may ask user to authorize certain website's access to the wallet and remember that choice. This is to limit spam attempts to connect to the wallets and possibly exploit vulnerabilities in them. Once allowed, browser will never ask user's permission and direct all request to the wallet. It's now wallet's job to get user's permission to give app access to the wallet.

### Native app proxy to hardware wallet

Similarly to web browser, native app may ask a permission to access hardware wallet just once. Then authorization happens on hardware wallet directly while native app simply transfers data to and from the requesting app.

Combination of these APIs enables a sophisticated chain of communication (web page with a hardware wallet) without any extra effort on anyone's part:

    Web page <-> JS API in browser <-> native API of a wallet client app <-> custom API of a hardware wallet

In this chain, web page accesses generic JS API. Web browser implements generic native extensions API (platform-dependent). Native app implements its own custom protocol to talk to its hardware wallet that implements Core Spec and authorizes transactions.


Error Handling
--------------

Error handling highly depends on actual implementation of this spec. But the general rule is: *Wallet should never leak private information* by offering a variety of error messages. In case of any error, App should receive a generic "failure" response while Wallet is free to tell the user actual reasons for error. 

Kinds of errors:

1. Wallet is not available. App receives "not available" error. App may provide user with alternative options to provide Bitcoins. Example: if [JS API](js_api_spec.md) cannot connect to any wallet, App may provide a [bitcoin: URL](url_spec.md) so user can go directly to his wallet app.

2. Not enough funds. Wallet may explain to user that App asks for more funds than user has. App sees generic "failure" error.

3. Authorization failed. Wallet may explain to user that his password is incorrect, fingerprint did not match, daily spending limit reached etc. App sees generic "failure" error.

4. No confirmed outputs available yet. Wallet may tell the user that his coins are not "mature" enough. App sees generic "failure" error.



