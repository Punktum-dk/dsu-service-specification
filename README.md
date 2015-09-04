# DSU Protocol 1.0

Simple DNSSEC interface at DK Hostmaster.

This manual describes version 1.0.

DSU is short for DS Update. DSU is DK Hostmaster's most simple interface for updating DNSSEC related DS records associated with a .dk domain name. The protocol is based on HTTP og the parameters are transferred as POST-variables. The answer contains an HTTP header and a brief message for human interpretation. The interface interprets a call as an atomic operation. If there is any errors, all changes are rejected and no existing DS records are deleted.

To use DSU, send an SSL-encrypted HTTP POST request to the address:

```
https://dsu.dk-hostmaster.dk/1.0
```

Non-ASCII parameters is first tried interpreted as UTF-8. If this fails, data is assumed to be ISO8859-1.

# Parameters

For the parameters defined further down, these rules apply:

An update can contain up to 5 DS keys per domain name.
If you wish to specify only 1 key, specify only one set, i.e. keytag1, algorithm1, digest_type1 and digest1.

If you wish to specify two keys, an additional set is specified, i.e. keytag2, algorithm2, digest_type2 and digest2. You may continue this way until you reach 5 sets.

The key sets must be specified sequentially starting from 1. E.g. it is not allowed to specify set 1, set 2, set 4 without also specifying set 3.

When a transaction is accepted, all previous DS keys associated with the domain name is deleted. This means that a transaction must contain all DS keys, which is to be associated with the domain name in the future.

The following parameters are part of the protocol:

**userid**

This userid must be authorised to make changes to the DS keys for the given domain name.

**password**

This is the password for the given userid.

**domain**

The domain name which this DS Update pertains. The domain name is transferred punycoded. This means so called æøå-names are written using the xn-- notation, just as in DNS.

**keytag1 .. keytag5**

The DNSKEY-key's keytag according to RFC4034 section 5.1.1: http://www.apps.ietf.org/rfc/rfc4034.html#sec-5.1.1.

**algorithm1 .. algorithm5**

The DNSKEY-ket's algorithm according to RFC5702 section 2: http://tools.ietf.org/html/rfc5702#section-2.

**digest_type1 .. digest_type5**

The digest method used to generate the DS fingerprint according to RFC4034 section 5.1.3: http://www.apps.ietf.org/rfc/rfc4034.html#sec-5.1.3.

**digest1 .. digest5**

The fingerprint digest of the DNSKEY-key according to RFC4509 section 2.1 http://www.apps.ietf.org/rfc/rfc4509.html#sec-2.1.

# Deleting DS-keys

If you wish to delete all DS-keys for a domain name, all values of set 1 is set to the value DELETE_DS. No further sets are allowed in the same transaction.

The reply is transferred primarily as HTTP status codes (http://www.iana.org/assignments/http-status-codes). A text message for human interpretation is also given. Possible status codes are:

200 - The request has been processed without problems.
400 - The request is invalid and has been rejected.
405 - The method used, is not allowed.
500 - An error occurred in DK Hostmaster's systems.
530 - Access denied.

If a 400-error is returned, the HTTP header will contain an additional error-code with the name X-DSU. The value can be one of the following:

480 - Userid not specified.
481 - Password not specified.
482 - Missing a parameter.
483 - Domain name not specified.
484 - Invalid domain name.
485 - Invalid userid.
486 - Invalid digest and digest_type combination.
487 - The contents of at least one parameter is syntactically wrong.
488 - At least one DS key has an invalid algorithm.
489 - Invalid sequence of sets.
495 - Unknown parameter given.
496 - Unknown userid.
497 - Unknown domain name.
See Example 2 below.

If a 530-error is returned, the HTTP header will contain an additional error-code with the name X-DSU. The value can be one of the following:

531 - Authentication failed.
532 - Authorisation failed.
533 - Authenticating using this password type is not supported.

# Example 1

Request (last line has been wrapped to increase the readability)

```
 POST /1.0 HTTP/1.0
 Host: dsu.dk-hostmaster.dk
 Content-Type: application/x-www-form-urlencoded
 Content-Length: 118

 userid=ABCD1234-DK&password=abba4evah&domain=a.dk&
 keytag1=DELETE_DS&algorithm1=DELETE_DS&digest_type1=DELETE_DS
 &digest1=DELETE_DS
Response

 HTTP/1.0 200 OK
 Content-Type: text/plain

 OK
```

# Example 2

Request (last line has been wrapped to increase the readability)

```
 POST /1.0 HTTP/1.0
 Host: dsu.dk-hostmaster.dk
 Content-Type: application/x-www-form-urlencoded
 Content-Length: 146

 userid=ABCD1234-DK&password=abba4evah&domain=xn--l-4ga.dk&
 keytag1=1551&algorithm1=7&digest_type1=1&
 digest1=CD1B87D20EE5EE5F78FCE25336E6519B838F7DC9
Response
 HTTP/1.0 400 Bad Request
 X-DSU: 496
 Content-Type: text/plain

 Unknown userid
 ```
