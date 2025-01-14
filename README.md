<a href="https://repology.org/project/wolfssl/versions">
    <img src="https://repology.org/badge/vertical-allrepos/wolfssl.svg" alt="Packaging status" align="right">
</a>

# wolfSSL Embedded SSL/TLS Library

The [wolfSSL embedded SSL library](https://www.wolfssl.com/products/wolfssl/) 
(formerly CyaSSL) is a lightweight SSL/TLS library written in ANSI C and
targeted for embedded, RTOS, and resource-constrained environments - primarily
because of its small size, speed, and feature set.  It is commonly used in
standard operating environments as well because of its royalty-free pricing
and excellent cross platform support. wolfSSL supports industry standards up
to the current [TLS 1.3](https://www.wolfssl.com/tls13) and DTLS 1.2, is up to
20 times smaller than OpenSSL, and offers progressive ciphers such as ChaCha20,
Curve25519, Blake2b and Post-Quantum TLS 1.3 groups. User benchmarking and
feedback reports dramatically better performance when using wolfSSL over
OpenSSL.

wolfSSL is powered by the wolfCrypt cryptography library. Two versions of
wolfCrypt have been FIPS 140-2 validated (Certificate #2425 and
certificate #3389). FIPS 140-3 validation is in progress. For additional
information, visit the [wolfCrypt FIPS FAQ](https://www.wolfssl.com/license/fips/)
or contact fips@wolfssl.com.

## Why Choose wolfSSL?

There are many reasons to choose wolfSSL as your embedded, desktop, mobile, or
enterprise SSL/TLS solution. Some of the top reasons include size (typical
footprint sizes range from 20-100 kB), support for the newest standards
(SSL 3.0, TLS 1.0, TLS 1.1, TLS 1.2, TLS 1.3, DTLS 1.0, and DTLS 1.2), current
and progressive cipher support (including stream ciphers), multi-platform,
royalty free, and an OpenSSL compatibility API to ease porting into existing
applications which have previously used the OpenSSL package. For a complete
feature list, see [Chapter 4](https://www.wolfssl.com/docs/wolfssl-manual/ch4/)
of the wolfSSL manual.

## Notes, Please Read

### Note 1
wolfSSL as of 3.6.6 no longer enables SSLv3 by default.  wolfSSL also no longer
supports static key cipher suites with PSK, RSA, or ECDH. This means if you
plan to use TLS cipher suites you must enable DH (DH is on by default), or
enable ECC (ECC is on by default), or you must enable static key cipher suites
with one or more of the following defines:

```
WOLFSSL_STATIC_DH
WOLFSSL_STATIC_RSA
WOLFSSL_STATIC_PSK
```
Though static key cipher suites are deprecated and will be removed from future
versions of TLS.  They also lower your security by removing PFS.

When compiling `ssl.c`, wolfSSL will now issue a compiler error if no cipher
suites are available. You can remove this error by defining
`WOLFSSL_ALLOW_NO_SUITES` in the event that you desire that, i.e., you're
not using TLS cipher suites.

### Note 2
wolfSSL takes a different approach to certificate verification than OpenSSL
does. The default policy for the client is to verify the server, this means
that if you don't load CAs to verify the server you'll get a connect error,
no signer error to confirm failure (-188).

If you want to mimic OpenSSL behavior of having `SSL_connect` succeed even if
verifying the server fails and reducing security you can do this by calling:

```c
wolfSSL_CTX_set_verify(ctx, WOLFSSL_VERIFY_NONE, NULL);
```

before calling `wolfSSL_new();`. Though it's not recommended.

### Note 3
The enum values SHA, SHA256, SHA384, SHA512 are no longer available when
wolfSSL is built with `--enable-opensslextra` (`OPENSSL_EXTRA`) or with the
macro `NO_OLD_SHA_NAMES`. These names get mapped to the OpenSSL API for a
single call hash function. Instead the name `WC_SHA`, `WC_SHA256`, `WC_SHA384` and
`WC_SHA512` should be used for the enum name.

# wolfSSL Release 5.4.0 (July 11, 2022)

Note:
** Future releases of wolfSSL will turn off TLS 1.1 by default
** Release 5.4.0 made SP math the default math implementation. To make an equivalent build as –disable-fastmath from previous versions of wolfSSL, now requires using the configure option –enable-heapmath instead.

Release 5.4.0 of wolfSSL embedded TLS has bug fixes and new features including:

## Vulnerabilities
* [High]  Potential for DTLS DoS attack. In wolfSSL versions before 5.4.0 the return-routability check is wrongly skipped in a specific edge case. The check on the return-routability is there for stopping attacks that either consume excessive resources on the server, or try to use the server as an amplifier sending an excessive amount of messages to a victim IP. If using DTLS 1.0/1.2 on the server side users should update to avoid the potential DoS attack. CVE-2022-34293
* [Medium] Ciphertext side channel attack on ECC and DH operations. Users on systems where rogue agents can monitor memory use should update the version of wolfSSL and change private ECC keys. Thanks to Sen Deng from Southern University of Science and Technology (SUSTech) for the report.
* [Medium] Public disclosure of a side channel vulnerability that has been fixed since wolfSSL version 5.1.0. When running on AMD there is the potential to leak private key information with ECDSA operations due to a ciphertext side channel attack. Users on AMD doing ECDSA operations with wolfSSL versions less than 5.1.0 should update their wolfSSL version used. Thanks to professor Yinqian Zhang from Southern University of Science and Technology (SUSTech), his Ph.D. student Mengyuan Li from The Ohio State University, and his M.S students Sen Deng and Yining Tang from SUStech along with other collaborators; Luca Wilke, Jan Wichelmann and Professor Thomas Eisenbarth from the University of Lubeck, Professor Shuai Wang from Hong Kong University of Science and Technology, Professor Radu Teodorescu from The Ohio State University, Huibo Wang, Kang Li and Yueqiang Cheng from Baidu Security and Shoumeng Yang from Ant Financial Services Group.
CVE-2020-12966 https://www.amd.com/en/corporate/product-security/bulletin/amd-sb-1013 CVE-2021-46744 https://www.amd.com/en/corporate/product-security/bulletin/amd-sb-1033


## New Feature Additions

### DTLS 1.3
* Support for using the new DTLSv1.3 protocol was added
* Enhancements to bundled examples for an event driven server with DTLS 1.3 was added
### Ports
* Update for the version of VxWorks supported, adding in support for version 6.x
* Support for new DPP and EAP-TEAP/EAP-FAST in wpa_supplicant
* Update for TSIP version support, adding support for version 1.15 for RX65N and RX72N
* Improved TSIP build to handle having the options WOLFSSL_AEAD_ONLY defined or NO_AES_CBC defined
* Added support for offloading TLS1.3 operations to Renesas RX boards with TSIP
### Misc.
* Constant time improvements due to development of new constant time tests
* Initial translation of API headers to Japanese and expansion of Japanese help message support in example applications
* Add support for some FPKI (Federal PKI) certificate cases, UUID, FASC-N, PIV extension for use with smart cards
* Add support for parsing additional CSR attributes such as unstructured name and content type
* Add support for Linux getrandom() when defining the macro WOLFSSL_GETRANDOM
* Add TLS 1.2 ciphersuite ECDHE_PSK_WITH_AES_128_GCM_SHA256 from RFC 8442
* Expand CAAM support with QNX to include i.MX8 boards and add AES-CTR support
* Enhanced glitching protection by hardening the TLS encrypt operations

## Math and Performance

### SP Math Additions
* Support for ARMv3, ARMv6 and ARMv7a
    - Changes and improvements to get SP building for armv7-a
    - Updated assembly for moving large immediate values on ARMv6
    - Support for architectures with no ldrd/strd and clz
* Reworked generation using common asm ruby code for 32bit ARM
* Enable wolfSSL SP math all by default (sp_int.c)
* Update SP math all to not use sp_int_word when SQR_MUL_ASM is available
### SP Math Fixes
* Fixes for constant time with div function
* Fix casting warnings for Windows builds and assembly changes to support XMM6-15 being non-volatile 
* Fix for div_word when not using div function
* Fixes for user settings with SP ASM and ED/Curve25519 small
* Additional Wycheproof tests ran and fixes
* Fix for SP math ECC non-blocking to always check `hashLen`
* Fix for SP math handling edge case with submod

## Improvements and Optimizations

### Compatibility Layer
* Provide access to "Finished" messages outside of compatibility layer builds
* Remove unneeded FIPS guard on wolfSSL_EVP_PKEY_derive
* Fix control command issues with AES-GCM, control command EVP_CTRL_GCM_IV_GEN
* Add support for importing private only EC key to a WOLFSSL_EVP_PKEY struct
* Add support for more extensions to wolfSSL_X509_print_ex
* Update for internal to DER (i2d) AIPs to move the buffer pointer when passed in and the operation is successful
* Return subject and issuer X509_NAME object even when not set
### Ports
* Renesas RA6M4 example update and fixes
* Support multi-threaded use cases with Renesas SCE protected mode and TSIP
* Add a global variable for heap-hint for use with TSIP
* Changes to support v5.3.0 cube pack for STM32
* Use the correct mutex type for embOS
* ESP-IDF build cleanup and enhancements, adding in note regarding ESP-IDF Version
* Support for SEGGER embOS and emNET
* Fix to handle WOLFSSL_DTLS macro in Micrium build
### Build Options
* Support for verify only and no-PSS builds updated
* Add the enable options wolfssh (mapped to the existing –enable-ssh)
* Remove WOLFSSL_ALT_NAMES restriction on notBefore/notAfter use in Cert struct
* Move several more definitions outside the BUILDING_WOLFSSL gate with linux kernel module build
* Modify --enable-openssh to not enable non-FIPS algos for FIPS builds
* Remove the Python wrappers from wolfSSL source (use pip install instead of using wolfSSL with Python and our separate Python repository)
* Add --enable-openldap option to configure.ac for building the OpenLDAP port
* Resolve DTLS build to handle not having –enable-hrrcookie when not needed
* Add an --enable-strongswan option to configure.ac for building the Strongswan port
* Improve defaults for 64-bit BSDs in configure
* Crypto only build can now be used openssl extra
* Update ASN template build to properly handle WOLFSSL_CERT_EXT and HAVE_OID_ENCODING
* Allow using 3DES and MD5 with FIPS 140-3, as they fall outside of the FIPS boundary
* Add the build option --enable-dh=const which replaces setting the macro WOLFSSL_DH_CONST and now conditionally link to -lm as needed
* Add the macro WOLFSSL_HOSTNAME_VERIFY_ALT_NAME_ONLY which is used to verify hostname/ip address using alternate name (SAN) only and does not use the common name
* WOLFSSL_DTLS_NO_HVR_ON_RESUME macro added (off by default to favor more security). If defined, a DTLS server will not do a cookie exchange on successful client resumption: the resumption will be faster (one RTT less) and will consume less bandwidth (one ClientHello and one HelloVerifyRequest less). On the other hand, if a valid SessionID is collected, forged clientHello messages will consume resources on the server.
* Misc.
* Refactoring of some internal TLS functions to reduce the memory usage
* Make old less secure TimingPadVerify implementation available
* Add support for aligned data with clang LLVM
* Remove subject/issuer email from the list of alt. Email names in the DecodedCerts struct
* Zeroizing of pre-master secret buffer in TLS 1.3
* Update to allow TLS 1.3 application server to send session ticket
* Improve the sniffer asynchronous test case to support multiple concurrent streams
* Clean up wolfSSL_clear() and add more logging
* Update to not error out on bad CRL next date if using NO_VERIFY when parsing
* Add an example C# PSK client
* Add ESP-IDF WOLFSSL_ESP8266 setting for ESP8266 devices
* Support longer sigalg list for post quantum use cases and inter-op with OQS's OpenSSL fork
* Improve AES-GCM word implementation of GMULT to be constant time
* Additional sanity check with Ed25519/Ed448, now defaults to assume public key is not trusted
* Support PSK ciphersuites in benchmark apps
* FIPS in core hash using SHA2-256 and SHA2-384
* Add ability to store issuer name components when parsing a certificate
* Make the critical extension flags in DecodedCert always available
* Updates to the default values for basic constraint with X509’s
* Support using RSA OAEP with no malloc and add additional sanity checks
* Leverage async code paths to support WANT_WRITE while sending packet fragments
* New azsphere example for continuous integration testing
* Update RSA key generation function to handle pairwise consistency tests with static memory pools used
* Resolve build time warning by passing in and checking output length with internal SetCurve function
* Support DTLS bidirectional shutdown in the examples
* Improve DTLS version negotiation and downgrade capability

### General Fixes
* Fixes for STM32 Hash/PKA, add some missing mutex frees, and add an additional benchmark
* Fix missing return checks in KSDK ED25519 code
* Fix compilation warnings from IAR
* Fixes for STM32U5/H7 hash/crypto support
* Fix for using track memory feature with FreeRTOS
* Fixup XSTR processing for MICRIUM
* Update Zephyr fs.h path
* DTLS fixes with WANT_WRITE simulations
* Fixes for BER use with PKCS7 to have additional sanity checks and guards on edge cases
* Fix to handle exceptional edge case with TFM mp_exptmod_ex
* Fix for stack and heap measurements of a 32-bit build
* Fix to allow enabling AES key wrap (direct) with KCAPI
* Fix --enable-openssh FIPS detection syntax in configure.ac
* Fix to move wolfSSL_ERR_clear_error outside gate for OPENSSL_EXTRA
* Remove MCAPI project's dependency on zlib version
* Only use __builtin_offset on supported GCC versions (4+)
* Fix for c89 builds with using WOLF_C89
* Fix 64bit postfix for constants building with powerpc
* Fixed async Sniffer with TLS v1.3, async removal of `WC_HW_WAIT_E` and sanitize leak
* Fix for QAT ECC to gate use of HW based on marker
* Fix the supported version extension to always check minDowngrade
* Fix for TLS v1.1 length sanity check for large messages
* Fixes for loading a long DER/ASN.1 certificate chain
* Fix to expose the RSA public DER export functions with certgen
* Fixes for building with small version of SHA3
* Fix configure with WOLFSSL_WPAS_SMALL
* Fix to free PKCS7 recipient list in error cases
* Sanity check to confirm ssl->hsHashes is not NULL before attempting to dereference it
* Clear the leftover byte count in Aes struct when setting IV

For additional vulnerability information visit the vulnerability page at:
https://www.wolfssl.com/docs/security-vulnerabilities/

See INSTALL file for build instructions.
More info can be found on-line at: https://wolfssl.com/wolfSSL/Docs.html



# Resources

[wolfSSL Website](https://www.wolfssl.com/)

[wolfSSL Wiki](https://github.com/wolfSSL/wolfssl/wiki)

[FIPS 140-2/140-3 FAQ](https://wolfssl.com/license/fips)

[wolfSSL Documentation](https://wolfssl.com/wolfSSL/Docs.html)

[wolfSSL Manual](https://wolfssl.com/wolfSSL/Docs-wolfssl-manual-toc.html)

[wolfSSL API Reference](https://wolfssl.com/wolfSSL/Docs-wolfssl-manual-17-wolfssl-api-reference.html)

[wolfCrypt API Reference](https://wolfssl.com/wolfSSL/Docs-wolfssl-manual-18-wolfcrypt-api-reference.html)

[TLS 1.3](https://www.wolfssl.com/docs/tls13/)

[wolfSSL Vulnerabilities](https://www.wolfssl.com/docs/security-vulnerabilities/)

[Additional wolfSSL Examples](https://github.com/wolfssl/wolfssl-examples)
