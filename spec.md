
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

2. **Bitcoin URL Extension** defines means to communicate core data structures using standard *bitcoin:* URL scheme. This allows simple, although not very transparent, integration between websites and native apps.

3. **Bitcoin JS API** defines JavaScript API for web browsers that allows web pages communicating to Bitcoin wallets.

4. **Bitcoin iOS/OSX extensions** defines a standard way to declare and access extensions provided by wallet apps on iOS and OS X (starting with iOS 8 and OS X 10.10).

5. **Bitcoin Android extentions** defines a standard way to declare extensions for Android apps.

6. Similar native APIs for Windows and Linux.



Core Spec
---------

Bitcoin Wallet API is based on three core protocols:

1. **Pay-to-Transaction**. App send an incomplete transaction to Wallet and asks to add a certain amount of coins in it. Wallet asks user's permission and adds signed inputs to the transaction.

2. **Inputs Authorization**. This is a two-step process. App requests from Wallet an authorization to spend certain amount of coins. Wallet asks user's permission and sends back to the app a list of unsigned inputs and change outputs. App uses them to compose a complete transaction that is sent back to Wallet for signing. Wallet signs the transaction if it correctly uses all authorized inputs and outputs.

3. **Key API**. App may outsource key storage to Wallet because it already implements security and safety measures such as encryption, authorization and backups. App can only access public keys specific to itself and request signatures of arbitrary data using these keys. Wallet never stores its own coins using these keys and only provides storage service to the app. Three requests are supported by this API: Public key request, ECDSA signature and Diffie-Hellman multiplication.


TODO: Define data structures:

1. Amount
2. Transaction
3. Input
4. Output
5. Signature script
6. Authorization token (includes inputs and outputs)

TODO: expand on exchange protocols



Bitcoin URL Extension
---------------------

This is extension of BIP 21 that defines *bitcoin:* URL format, optional and required fields, backward and forward compatibility.

1) **Pay-to-Transaction URL**

    bitcoin:[address]?tx=<base58check encoding of a transaction>&amount=<btc>[&message=...]
    
Wallet asks user's permission to spend certain amount in that transaction. Then it adds necessary inputs and outputs, signs transaction and broadcasts it.

Note: if the app supports funding via its own temporary wallet, it may also provide payment address before "?" which will be used by wallets not supporting the API.

Note: use *message* parameter to explain the reason for request to the user.


2) **Inputs Authorization**

Step 1: authorize inputs.

    bitcoin:[address]?authinputs=<url>&amount=<btc>[&message=...]

Wallet asks user's permission to authorize a given amount. Then it prepares unsigned inputs and change outputs and sends them via the given URL.

App may also provide a bitcoin address as a fallback for wallets that do not support this API.

Step 2: sign a transaction.

    bitcoin:?tx=<base58check encoding of a transaction>&req-token=<base58check encoding of an authorization token>[&req-return=<url where to post the signed transaction>]

Wallet checks if the token is valid and all its inputs and outputs are present in a given transaction. If correct, it inserts the signatures and broadcasts the transaction.

If *req-return* is present, instead of broadcasting, wallet calls the given URL with the base64url-encoded signatures for the transaction.

Note: use *message* parameter to explain the reason for request to the user.

TODO: we need to define a way for app to explain a reason for payment.

3) **Key API**

**Public Key request:**

    bitcoin:?&req-pubkey-index=<key index>&id=<base58check compressed pubkey>&t=<timestamp>&return=<url where to return the pubkey>[&message=...]&sig=<signature>


**Signature API:**

Sometimes in case of web API or native extension API receiver may authenticate the caller using system-provided identifier. E.g. domain name (www.example.com) in JS API or bundle identifier (com.example.myapp) in iOS extensions. 

In cases when secure identification is not possible, App may identify itself simply by a public key and prove itself using a one-time signature of bitcoin: request.

    bitcoin:?req-sign-hash=<hex hash to sign>&i=<key index>&id=<base58check compressed pubkey>&t=<timestamp>&return=<url where to return the signature>[&message=...]&sig=<signature>

*req-sign-hash*: hash to sign. Hash must be minimum 160 bits long and maximum 512 bits long. Only this parameter is marked "req-" to make sure incompatible wallets fail. Other parameters (i, id, t, sig) are required.

*i*: index of a key to be used that is derived from a root key.

*id*: pubkey identifying the app. It is used to derive the root key.

*t*: UNIX timestamp of the current time. Wallet should ignore too old URLs or warn users about them. Recommended to only allow URLs within last 60 seconds and ignore any older or future ones.

*return*: URL where to return the signature in Base58Check format.

Wallet produces a key by mixing the root key with a given pubkey (DH) after validating that the bitcoin: request is not too old (less than a minute since receiving the request) and is properly signed by a given public key.

Source key is supposed to be stored in a secure location.

In some contexts this API may be of a no value. E.g. if the secure storage of an identifying public key may also be used for storing actual keys that user is supposed to use within the App. But in server-client Apps where server keeps the key and client authorizes transactions, this API is useful.

**Diffie-Hellman API:**

    bitcoin:?req-dh=<base58check pubkey to multiply>&i=<key index>&id=<base58check compressed pubkey>&t=<timestamp>&return=<url where to return the signature>[&message=...]&sig=<signature>

Instead of signing a hash, multiplies a given public key by a derived private key and returns the resulting public key.



Bitcoin JS API
--------------


TODO: write up on JS functions, structures and callbacks for each of the APIs.

Note 1: Wallet must return a list of signatures, but JS API should put them automatically in user's transaction. So the app may simply trust the transaction but not trust the wallet.

Note 2: Key API should support 3 functions: pubkey request for a given index, ECDSA signature for a given hash and DH pubkey for a given pubkey.





iOS and OS X extensions
-----------------------

TODO: Define extensions identifiers, data structure wrappers and a common Objective-C/Swift wrapper for using them.



Android extensions
------------------

TODO: Define extensions identifiers, data structure wrappers and a common Java wrapper for using them.



Windows extensions
------------------

TODO: Define extensions identifiers, data structure wrappers and a common .NET/C# wrapper for using them.


