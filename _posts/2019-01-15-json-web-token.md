---
layout: post
tags: [aws, rails, heroku]
bigimg: /img/posts/2019-01-15-json-web-token/jwt.jpg
---

JSON Web Tokens(JWT) is a useful standard because it sends information that can be verified and trusted with a digital signature. It is used for transferring secure data as JSON objects. They are also means of storing data that’s easy to decode and encode.

Uses of JWT can be in Authentication and Information exchange.

It consists of three parts –
1. Header – It tells how the signature key should be computed, i.e. using which algorithm.
2. Payload(Data we want to extract)
3. Verify signature – computed using the algorithm stated in header the parameters include the data in payload section and a secret key.

These three sections are separated by (.) – (header.payload.signature).

## How JWT works?

![how-jwt-works](/img/posts/2019-01-15-json-web-token/how-jwt-works.png)

We have a client and a server. Client wants to access data from server, but server knows that **client cannot be trusted**. Server only wants to give data to a trustworthy client.

So our client sends a request to our server along with data to verify who it is. Now server checks the data validity and accepts or rejects it.

If the data is accepted then instead of saving this data the server instead creates a token. This token is then returned to the client. Now it’s up to client to store this data send it along as required for any request to the server in future.

The next time our client makes a request along a secure route, it does just that, it sends along the authentication token.
But our server knows **client cannot be trusted**. So our server verifies this token is who it says its from and that it hasn’t been tampered.

If everything is fine then server sends a response with the requested data. 

**Remember -**  The purpose of using JWT is not to hide or secure data in any way. JWT are used just to prove that the sent data was actually created by an authentic source.

