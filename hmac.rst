.. _hmac:


.. toctree::
   :maxdepth: 2
   :caption: API Authentication

REST Authentication
===================
Client applications must pass authentication information with all API calls.
The IAMPASS libraries have this functionality built into them and make integration easier.
This document describes the authentication data required.



Authentication Protocol 1
-------------------------

In addition to connections be encrypted, they must also be authenticated. Authentication is performed using the
Hash Message Authentication Code (HMAC) in conjunction with the SHA-256 algorithms as defined RFC 4868,
specifically HMAC-SHA-256-128.

The HTTP Authentication Header is used to define the credentials. Further,
additional headers are used to increase entropy, limit the life space of a HMAC and also link an specific authentication to the requested URL.


One-time-use Token
^^^^^^^^^^^^^^^^^^
IAMPASS will assign each client application a unique client ID and a 192 bit (24 byte) Shared Secret.

The token is derived by creating a 64 bit (8 byte) nonce and pre-pending it to the 192 bit Shared Secret to create a 256 bit key.

The nonce must not be reused within a reasonable amount of time.

Further, the nonce should not be constructed only from printable characters. All 64 bits should have equal probability of being set to a 0 or 1.

A non-normative method for doing this is to use a pseudo-random number generator that is properly seeded with sufficient entropy and choose an unsigned number between 0 and 264 – 1. Section 2.1.1 of RFC 4868 prohibits the use of keys that are not identically 256 bit. Authentication will be rejected if the nonce is not 64 bits.

The 256 bit key (nonce + Shared Secret) are passed to a SHA-256 function. This will produce a 256 bit output and the left most 128 bits (see RFC 2104) are selected as the token.

In pseudo-code the procedure to produce the token would be:

.. code-block::

    nonce = PRG(0, 264 – 1, uniform distribution)
    key = concatenate(nonce, Shared Secret)
    SHA256_Output = SHA-256(key)
    token = left_truncate(SHA256_Output, 128 bits)

HMAC Signature
^^^^^^^^^^^^^^

After computing the token, the HMAC signature needs to be determined. This is done in a three step process. First the key is computed, followed by the digest and then the signature.
This key consists of the concatenation of nonce, the Request URI and the authentication time stamp (represented as a string). The nonce must be represented as a string without any leading zeros removed as it must represent 64 bits of data. The Request URI must be exactly what is in the header, including scheme, resource and query string. The pseudo-code for producing the key:
key = concatenate(string(nonce), Request URI, authentication time stamp)
A non-normative example would be:

.. code-block:: python

    nonce = 9223372036854775807
    application ID = ABCD
    request_uri = https://main.iam-api.com/management/add_users/ABCD
    time_stamp = 1234567890
    key = string(nonce) + request_uri + string(time_stamp)

This would yield:
key=9223372036854775807https://main.iam-api.com/management/add_users/ABCD


The digest is determined by computing the HMAC-SHA-256 using the token as the secret and the above key. This digest is than truncated to the left most 128 bits. The pseudo-code to compute the
digest is:
digest_256=HMAC-SHA-256(token, key)
digest = left_truncate(digest_256, 128 bits)
The final step is to determine the signature. The signature is the base-64 encoded representation of the digest. In pseudo-code this would be:
signature = base64encode(digest)

HTTP Headers
^^^^^^^^^^^^

Several headers are used to transmit the parameters needed for the authenticating system to reconstruct the digest it was sent. These headers are also used to define what authentication scheme is being used to create the HMAC digest, limit the time frame that a specific HMAC can be used.
The time at which the API authentication request is being made is encoded in a header. The format of the time stamp is the number of seconds since January 1, 1970 00:00:00 GMT; known as ctime or linux time. The X-IAMPASS-Authentiaction-Timestamp header is used. The following is a non-normative example where 1234567890 represents the time of the request.
X-IAMPASS-Authentiaction-Timestamp: 1234567890
The X-IAMPASS-Authentiaction-Version header is used to represent the version of the
authentication scheme that is being used. The version described in this section would have the header:
X-IAMPASS-Authentiaction-Version: 1
The Authentication header consists of the authentication scheme, hmac, and the authentication data. The authentication data has three components that are separated by a colon and no white space. The first portion is the identifier used by the client. The second component is the nonce used in generating the token and the signature and it must be converted from an integer to a string. Care must be taken that any leading zeros are not lost as the nonce must represent 64 bits. The final section is the signature. It has the following format:
1
Authentication: hmac client:nonce:signature
