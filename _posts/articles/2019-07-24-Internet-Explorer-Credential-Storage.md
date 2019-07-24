---
published: false
layout: post
categories: articles
tags:
  - windows
---
# Summary
{:.no_toc}

* toc
{:toc}

# Overview

Research has shown that Internet Explorer can store (and recall) various pieces of information in a manner that can be convenient for a user.  These pieces of information can be loosely broken into a few different categories:

  HTTP Authentication credentials (e.g., those creds used when a server makes use of say, the HTTP Basic Authentication scheme)
    FTP credentials
    Autocomplete Data, which can be broken down further into two sub categories:
        Form Data
        Password Data

The exact implementation details for storage and retrieval will, of course, vary based on the category of information, version of Internet Explorer, and potentially, the version of the operating system.  As information is discovered about these mechanisms, it is encouraged that the information be recorded here for the benefit of others.

# Discussion

## Autocomplete Password Data

Internet Explorer will store and retrieve what it believes to be login-style data in a manner that is convenient for a user.  Research, along with empirical testing shows that this data is commonly stored as paired strings such as that for a username/password combination.  IE applies some heuristics in order to decide whether to treat certain form data as autocomplete-worthy and whether to treat said data as password/login information(1).  The way in which IE processes login data has undergone some changes over the years.  The information that follows is best broken into the following sections:

   Internet Explorer versions 7-9
   Internet Explorer versions 10+

### IE7-9

IE will store Autocomplete Password data and Autocomplete Form data in a similar manner s(3), (4), (5), (6), (10), (11).  Autocomplete Password data will be stored in the registry under the following key:

as an example, the following simple HTML will appear to be a login form to IE.

Note the 'type=password' attribute & value for the input element named 'password'.  One can use common utilities such as procmon or regshot to see IE interact with the 'Storage2' registry key.

In the aforementioned registry key, Internet Explorer creates REG_BINARY entries, each of which correspond to a visited URL associated with the Autocomplete data.  The Value Name of a REG_BINARY value is the hex representation of a combination of the hex representation of the 20 byte SHA1 has of the URL, along with a "checksum" byte.  The value name is crafted via:

   1.For a given URL, which is in lowercase (unknown what that would actually mean for a non-western alphabet) and encoded as a NULL terminated, UTF-16 LE string, calculate the SHA-1 hash of the URL.  This will yield a twenty byte value.
   2.For each byte of the twenty bytes of the URL's hash, add them together modulo 256 (i.e., store a running tally of the bytes of the hash in an unsigned char)
   3.Create a string which consists of the concatenation of the hex representation of the hash and the hex representation of the checksum byte's value.

For an example, consult the following Python (2.7.x) session/code:
