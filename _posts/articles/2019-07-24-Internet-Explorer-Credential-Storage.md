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

~~~
>>> import hashlib
>>> url = "http://192.168.146.128:8080/creds_form_savepass.html" + "\x00"
>>> url_utf_16_le = url.encode("utf-16-le")
>>> sha1obj = hashlib.sha1(url_utf_16_le)
>>> urldigest = sha1obj.digest()
>>> checksum = 0
>>> len(urldigest)
20
>>> for abyte in urldigest:
...   checksum = (checksum + (ord(abyte))) & 0xFF
...
>>> hash = sha1obj.hexdigest().upper()
>>> cksum = "%02X" % checksum
>>> reg_value_name = "%s%s" % (hash, cksum)
>>> print reg_value_name
263E55A1AC93C5F70F2F3CAB9F1DCEE4A899B2E9C4
~~~

the resultant reg_value_name is the value name which will be found in the registry under the aforementioned registry key.

Credential information is stored in an encrypted form in the Value of the REG_BINARY entry.   In order to decrypt the data for a particular entry, one needs the original URL which was used to create the entry (i.e., the URL which was hashed to create the Value Name), and the creds of the user to whom that data belongs (i.e., the currently logged in user).  Assuming that the user who (indirectly) caused the REG_BINARY entry to be created is currently logged in, the (lowercase, but again, what does that mean for non-western Unicode code units??) utf-16-le NULL terminated string representation of the URL is used to create the DATA_BLOB which is passed as the pOptionalEntropy parameter in a call to the CryptUnprotectData function.

Once the binary value has been decrypted, all that remains is to parse out the credentials of interest.  Microsoft apparently uses some undocumented structures in order to serialize and de-serialize the data from the REG_BIANRY entry in the registry.  The magic happens in IEFRAME.DLL and the magic object is a CStringList.  If you are following along with us at home, you can go grab symbols, pop open IDA or windbg, and look for the CStringList::ReadFromBlob and CStringList::WriteToBlob as a starting point for your reversing and verification efforts.  There is also a "signature" value of 0x4b434957 which is also good to search for while reviewing the IDA idb of IEFRAME.DLL.  Further, some good initial, yet apparently slightly outdated starting points to undertstanding the structs can be found in at least one of the references below(13).

Inferred (and apparently up-to-date) structure definitions (NOT ripped from current source) can be seen in internal code which is designed for review and customization 
.  A brief overview is given below to assist one in visualizing the binary format of the decrypted data:

Structure names given are those which are found in the aforementioned internal code.  Interesting things to note are:

   Each STRINGENTRY corresponds to a given string
   Matched strings (e.g., from a username/password) combo will have matched FILETIME values as expressed in the strings' corresponding STRINGENTRYs
   There are no separators in the string data, just one NULL terminated string after another.

### IE10+

Research indicates that the means used to process login data can now vary based on the underlying operating system.  As noted(2), starting with IE10, if IE is installed on Windows 8, the login data is no longer stored in the registry, but in the Credential Manager.  Experimental evidence indicates that IE11 will still store login data in the registry (as in IE versions 7-9), if installed and running on a Windows 7 box.  Thus, your mileage may vary based on the combination of browser version and underlying operating system.

More information is to be added in this section.  Until then, feel free to browse some of the references such as, (6) and (10).

## Autocomplete Form Data

Information to be added.  Autocomplete Form data is accessed in a manner very similar to that of Autocomplete Password data.  The registry key used to store form data is:

	HKCU\Software\Microsoft\Internet Explorer\IntelliForms\Storage1
    
Note that reference (12) indicates that the pOptionalEntropy parameter (which acts as part of the decryption key) will differ depending upon the type of data.  Additionally. reference (12) mentions that for Autocomplete FORM data, the "name" attribute of the corresponding HTML input element is used as the "optional entropy", in a manner similar to how the URL was used for the Autocomplete Password data as specified above.

Some sample HTML which will cause IE to use the Autocomplete Form storage is shown below:

~~~
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
  <head>
      <title>
        A Form to test cred gathering
      </title>
  </head>
  <body>
    <form method=GET action="">
        <p>Username:<input type=text name=username size=32 maxlength=32>
        <p>Password:<input type=text name=password size=32 maxlength=32>
        <p><input type=reset value="RESET">
        <p><input type=submit name="submitit" value="SUBMIT">
    </form>
  </body>
</html>
~~~

