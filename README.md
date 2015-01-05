Name
====

dnscrypt-wrapper - A server-side dnscrypt proxy.

(c) 2012-2015 Yecheng Fu <cofyc.jackson at gmail dot com>

[![Build Status](https://travis-ci.org/Cofyc/dnscrypt-wrapper.png?branch=master)](https://travis-ci.org/Cofyc/dnscrypt-wrapper)

Description
===========

This is dnscrypt wrapper (server-side dnscrypt proxy), which helps to
add dnscrypt support to any name resolver.

This software is modified from
[dnscrypt-proxy](https://github.com/jedisct1/dnscrypt-proxy).

Installation
============

Install [libsodium](https://github.com/jedisct1/libsodium) and libevent2 first.

On Linux:

    $ ldconfig # if you install libsodium from source
    $ git clone --recursive git://github.com/Cofyc/dnscrypt-wrapper.git
    $ make configure
    $ ./configure
    $ make install

On FreeBSD:

    $ pkg_add -r gmake autoconf
    $ pkg_add -r libevent2
    $ gmake LDFLAGS='-L/usr/local/lib/event2 -L/usr/local/lib' CFLAGS=-I/usr/local/include

On OpenBSD:

    $ pkg_add -r gmake autoconf
    $ pkg_add -r libevent
    $ gmake LDFLAGS='-L/usr/local/lib/' CFLAGS=-I/usr/local/include/

Usage
=====

First, generate provider keypair:

    # stored in public.key/secret.key in current directory
    $ ./dnscrypt-wrapper --gen-provider-keypair

Second, generate crypt keypair:

    # stored in crypt_public.key/crypt_secret.key in current directory
    $ ./dnscrypt-wrapper --gen-crypt-keypair

Third, generate pre-signed certificate (use pre-generated key pairs):

    # stored in dnscrypt.cert in current directory
    $ ./dnscrypt-wrapper --crypt-secretkey-file crypt_secret.key --crypt-publickey-file=crypt_public.key --provider-publickey-file=public.key --provider-secretkey-file=secret.key --gen-cert-file

Run the program with pre-signed certificate:

    $ ./dnscrypt-wrapper  -r 8.8.8.8:53 -a 0.0.0.0:443  --crypt-secretkey-file=crypt_secret.key --crypt-publickey-file=crypt_public.key --provider-cert-file=dnscrypt.cert --provider-name=2.dnscrypt-cert.yechengfu.com

If you can store generated pre-signed certificate (binary string) in TXT record for your provider name, for example: 2.dnscrypt-cert.yourdomain.com. Then you can omit `--provider-cert-file` option. Name server will serve this binary certificate data for you.

P.S. We still provide `--provider-cert-file` option, because it's not convenient to store such long binary data in dns TXT record sometimes. But it's easy to configure it in your own dns servers (such as tinydns, etc). `--gen-cert-file` will generate example record in stdout.

Run dnscrypt-proxy to test against it:

    # --provider-key is public key fingerprint in first step.
    $ ./dnscrypt-proxy -a 127.0.0.1:55 --provider-name=2.dnscrypt-cert.yechengfu.com -r 127.0.0.1:443 --provider-key=<provider_public_key_fingerprint>
    $ dig -p 55 google.com @127.0.0.1

`<provider_public_key_fingerprint>` is public key fingerprint generated by `./dnscrypt-wrapper --gen-provider-keypair`, e.g. 4298:5F65:C295:DFAE:2BFB:20AD:5C47:F565:78EB:2404:EF83:198C:85DB:68F1:3E33:E952.

Optional, add `-d/--daemonize` flag to run as daemon.

Run `./dnscrypt-wrapper -h` to view command line options.

See also
========
    
- http://dnscrypt.org/
- http://www.opendns.com/technology/dnscrypt/
