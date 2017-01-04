# DSU Service and Protocol 1.0 Specification

<!-- MarkdownTOC -->

- [Introduction](#introduction)
- [About this Document](#about-this-document)
    - [License](#license)
    - [Document History](#document-history)
- [The .dk Registry in Brief](#the-dk-registry-in-brief)
- [The DS Update Service](#the-ds-update-service)
    - [Adding DS-keys](#adding-ds-keys)
    - [Deleting DS-keys](#deleting-ds-keys)
    - [Example 1](#example-1)
    - [Example 2](#example-2)
- [References](#references)
- [Resources](#resources)
    - [Mailing list](#mailing-list)
    - [Issue Reporting](#issue-reporting)
    - [Demo Client](#demo-client)
- [Appendices](#appendices)
    - [HTTP Status Codes](#http-status-codes)

<!-- /MarkdownTOC -->

<a name="introduction"></a>
# Introduction

DSU is short for DS Update. DSU is a propriety protocol and service developed and offered by DK Hostmaster's as an interface for updating DNSSEC related DS records associated with a .dk domain name. 

The protocol is based on HTTP og the parameters are transferred as POST-variables. The response contains an HTTP header and a brief message for human interpretation. The interface interprets a call as an atomic operation. If errors occur, all changes are rejected and no existing DS records are deleted.

To use DSU, send an SSL-encrypted HTTP POST request to the following address:

```
https://dsu.dk-hostmaster.dk/1.0
```

Non-ASCII parameters is first tried interpreted as UTF-8. If this fails, data is assumed to be ISO8859-1.

<a name="about-this-document"></a>
# About this Document

This specification describes protocol version 1.0.

Printable version can be obtained via [this link](https://gitprint.com/DK-Hostmaster/dsu-service-specification/blob/master/README.md), using the gitprint service.

<a name="license"></a>
## License

This document is copyright by DK Hostmaster A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a name="document-history"></a>
## Document History

* 1.1 2016-06-29
  * Added more information and extended the documentation with license, TOC etc.

* 1.0 2015-07-04
  * Initial revision on Github

<a name="the-dk-registry-in-brief"></a>
# The .dk Registry in Brief

DK Hostmaster is the registry for the ccTLD for Denmark (dk). The current model used in Denmark is based on a sole registry, with DK Hostmaster maintaining the central DNS registry.

The service is not subject to any sorts of standards.

<a name="the-ds-update-service"></a>
# The DS Update Service

<a name="adding-ds-keys"></a>
## Adding DS-keys

For the parameters defined further down, these rules apply:

An update can contain up to 5 DS keys per domain name.
If you wish to specify only 1 key, specify only one set, i.e. `keytag1`, `algorithm1`, `digest_type1` and `digest1`.

If you wish to specify two keys, an additional set is specified, i.e. `keytag2`, `algorithm2`, `digest_type2` and `digest2`. You may continue this way until you reach the maximum of 5 sets.

The key sets must be specified sequentially starting from 1. E.g. it is not allowed to specify set 1, set 2, set 4 without also specifying set 3.

When a transaction is accepted, all previous DS keys associated with the domain name is deleted. This means that a transaction must contain all DS keys, which is to be associated with the domain name in the future.

The following parameters are part of the protocol:

**userid**

This userid must be authorised to make changes to the DS keys for the given domain name.

**password**

This is the password for the given userid.

**domain**

The domain name which this DS Update pertains. The domain name is transferred encoded using punycode. This means domain name containing danish letters should be written using the `xn--` notation, just as for DNS. For allowed characters please see [the DK Hostmaster Name Service specification][DKHMDNSSPEC].

**keytag1 .. keytag5**

The DNSKEY-key's keytag according to [RFC:4034][RFC4034] [section 5.1.1][RFC4034_sec_5_1_1].

**algorithm1 .. algorithm5**

The DNSKEY-key's algorithm according to [RFC:5702][RFC5702] [section 2][RFC5702_sec_2] for algorithms 8 and 10 and [RFC:6605][RFC6605] for algorithms 13 and 14.

**digest_type1 .. digest_type5**

The digest method used to generate the DS fingerprint according to [RFC:4034][RFC4034] [section 5.1.3][RFC4034_sec_5_1_3]

**digest1 .. digest5**

The fingerprint digest of the DNSKEY-key according to [RFC:4509][RFC4509] [section 2.1][RFC4509_sec_2_1] or [RFC:6605][RFC6605] for digest type 4.

<a name="deleting-ds-keys"></a>
## Deleting DS-keys

If you wish to delete all DS-keys for a domain name, all values of set 1 is set to the value `DELETE_DS`. No further sets are allowed in the same transaction.

See Example 2 below.

If a `530` error is returned, the HTTP header will contain an additional error-code with the name `X-DSU`. The value can be one of the following:

- `531` Authentication failed.
- `532` Authorisation failed.
- `533` Authenticating using this password type is not supported.

<a name="example-1"></a>
## Example 1

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

<a name="example-2"></a>
## Example 2

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

<a name="references"></a>
# References

- [RFC:4034][RFC4034]
- [RFC:4509][RFC4509]
- [RFC:5702][RFC5702]
- [DK Hostmaster Name Service Specification][DKHMDNSSPEC]

<a name="resources"></a>
# Resources

Resources for DK Hostmaster DSU support are listed below.

<a name="mailing-list"></a>
## Mailing list

DK Hostmaster operates a mailing list for discussion and inquiries  about the DK Hostmaster DSU service and DNSSEC ingeneral. To subscribe to this list, write to the address below and follow the instructions. Please note that the list is for technical discussion only, any issues beyond the technical scope will not be responded to, please send these to the contact issue reporting address below and they will be passed on to the appropriate entities within DK Hostmaster A/S.

* `tech-discuss+subscribe@liste.dk-hostmaster.dk`

<a name="issue-reporting"></a>
## Issue Reporting

For issue reporting related to this specification, the DSU implementation or sandbox or production environments, please contact us. You are of course welcome to post these to the mailing list mentioned above, otherwise use the address specified below:

 * `info@dk-hostmaster.dk`

<a name="demo-client"></a>
## Demo Client

A [demo client](https://github.com/DK-Hostmaster/dsu-demo-client-mojolicious) is available as open source under a MIT license. 

<a name="appendices"></a>
# Appendices

<a name="http-status-codes"></a>
## HTTP Status Codes

The reply is transferred primarily as HTTP status codes (http://www.iana.org/assignments/http-status-codes). A text message for human interpretation is also provided. Possible status codes are:

| HTTP Status code | Message | Description |
|-------------|---------|-------------|
| 200 | OK | The request has been processed without problems |
| 400 | Bad Request | The request is invalid and has been rejected |
| 405 | Method Not Allowed | The method `POST` or `GET`, is not allowed |
| 500 | Internal Server Error | An error occurred in DK Hostmaster's systems |
| 530 | Access denied | Authentication not successful |

If a `400` error is returned, the HTTP header will contain an additional error code with the name `X-DSU`. The value can be one of the following:

| X-DSU Status code | Description |
|-------------|-------------|
| 480 | Userid not specified |
| 481 | Password not specified |
| 482 | Missing a parameter |
| 483 | Domain name not specified |
| 484 | Invalid domain name |
| 485 | Invalid userid |
| 486 | Invalid digest and digest_type combination |
| 487 | The contents of at least one parameter is syntactically wrong |
| 488 | At least one DS key has an invalid algorithm |
| 489 | Invalid sequence of sets |
| 495 | Unknown parameter given |
| 496 | Unknown userid |
| 497 | Unknown domain name |

[RFC4034]: http://tools.ietf.org/html/rfc4034
[RFC4034_sec_5_1_1]: https://tools.ietf.org/html/rfc4034#section-5.1.1
[RFC4034_sec_5_1_3]: https://tools.ietf.org/html/rfc4034#section-5.1.3
[RFC4509]: http://tools.ietf.org/html/rfc4509
[RFC4509_sec_2_1]: https://tools.ietf.org/html/rfc4509#section-2.1
[RFC5702]: http://tools.ietf.org/html/rfc5702
[RFC5702_sec_2]: https://tools.ietf.org/html/rfc5702#section-2
[RFC6605]: https://tools.ietf.org/html/rfc6605
[DKHMDNSSPEC]: https://github.com/DK-Hostmaster/dkhm-name-service-specification#domain-names
