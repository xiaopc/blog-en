---
title: Introduction for Crypto-JS and a common issue
id: 33
categories:
  - frontend
date: 2020-05-29 09:07:36

---

*original post date: July 26, 2016*

## Overview

Using JS encryption is a viable option when HTTPS is not available, or when front-end security needs to be increased.

Crypto-js supports Hashes like MD5/SHA-1/SHA-2/SHA-3/HMAC/PBKDF2, and ciphers like AES/DES/3DES/Rabbit/RC4/RC4Drop, with optional Block Modes and Padding. It can load a single mode only.

Link: [https://code.google.com/archive/p/crypto-js/](https://code.google.com/archive/p/crypto-js/) &amp;&amp; [https://github.com/brix/crypto-js](https://github.com/brix/crypto-js)

<!--more-->

## MD5

`md5.js` can be referenced separately.

{% codeblock lang:javascript %}
CryptoJS.MD5("test");
{% endcodeblock %}

## PBKDF2

`pbkdf2.js` can be referenced separately.

{% codeblock lang:javascript %}
// official examples
var str = '123456';
var salt = CryptoJS.lib.WordArray.random(128/8);

var key128Bits = CryptoJS.PBKDF2(str, salt, { keySize: 128/32 });
var key256Bits = CryptoJS.PBKDF2(str, salt, { keySize: 256/32 });
var key512Bits = CryptoJS.PBKDF2(str, salt, { keySize: 512/32 });

var key512Bits1000Iterations = CryptoJS.PBKDF2("Secret Passphrase", salt, {
keySize: 512/32,
iterations: 1000
});
{% endcodeblock %}

## AES

The default is AES-CBC-Pkcs5 (PKcs7), while other ciphers can be referenced separately. The key length is automatically determined with the length of the password entered. If using [Passphrase(https://en.wikipedia.org/wiki/Passphrase)], it's 256 bits.

Encryption: (here's the issue mentioned before)

{% codeblock lang:javascript %}
var str = '123456'; // content to encrypt
var key = '0123456789abcdef'; // key
var iv = '0123456789abcdef'; // initial iv (default is a random number)

key = CryptoJS.enc.Utf8.parse(key); // **Attention: if jQuery is loaded, add "toString()"**
iv = CryptoJS.enc.Utf8.parse(iv); // **same as above**

var encrypted = CryptoJS.AES.encrypt(str, key, {
 iv: iv,
 mode: CryptoJS.mode.CBC,
 padding: CryptoJS.pad.Pkcs7
});

// **encrypt() returns an object!**
encrypted = encrypted.toString();
{% endcodeblock %}

Decryption：
{% codeblock lang:javascript %}
var decrypted = CryptoJS.AES.decrypt(encrypted, key, {
 iv: iv,
 mode: CryptoJS.mode.CBC,
 padding: CryptoJS.pad.Pkcs7
});

// to utf-8 string
decrypted = CryptoJS.enc.Utf8.stringify(decrypted);
{% endcodeblock %}

## Other Ciphers

It's basically the same, except that Rabbit and RC4 don't support defining mode and padding.

* * *

References:

[1] https://blog.zhengxianjun.com/2015/05/javascript-crypto-js/

[2] http://stackoverflow.com/questions/35529804/using-crypto-js-to-encrypt-password-and-send-form-via-ajax-and-decrypt-in-java
