.. _keyhandling:

How to deal with Cryptographic keys
===================================

Absolutely vital to doing cryptography is to be able to handle keys.
This document will show you have to accomplish this with this package.

CryptoJWT deals with keys by defining 4 'layers'.

    1. At the bottom we have the keys as used by the cryptographic package
       (in this case cryptography_) .
    2. Above that we have something we call a JSON Web Key. The base class
       here is :py:class:`cryptojwt.jwk.JWK`. This class can import keys in
       a number of formats and can export a key as a JWK_.
    3. A :py:class:`cryptojwt.key_bundle.KeyBundle` keeps track of a set of
       keys that has the same origin. Like being part of a JWKS_.
    4. A :py:class:`cryptojwt.key_jar.KeyJar` lastly is there to keep the keys
       sorted by their owners/issuers.


I will not describe how to deal with keys in layer 1, that is done best by
cryptography_. So, I'll start at layer 2.

JSON Web Key (JWK)
------------------

Let us start with you not having any key at all and you want to create a
signed JSON Web Token (JWS_).
What to do ?

Staring with no key
...................

Well if you know what kind of key you want, and if it is a asymmetric key you
want, you can use one of the provided factory methods.

    RSA
        :py:func:`cryptojwt.jwk.rsa.new_rsa_key`
    Elliptic Curve:
        :py:func:`cryptojwt.jwk.ec.new_ec_key`


As an example::

    >>> from cryptojwt.jwk.rsa import new_rsa_key
    >>> rsa_key = new_rsa_key()
    >>> type(rsa_key)
    <class 'cryptojwt.jwk.rsa.RSAKey'>


If you want a symmetric key you only need some sort of "secure random"
mechanism. You can use this to acquire a byte array of the appropriate length
(e.g. 32 bytes for AES256), which can be used as a key.

When you have a key in a file on your hard drive
................................................

If you already have a key, like if you have a PEM encoded private RSA key in
a file on your machine you can load it this way::

    >>> from cryptojwt.jwk.rsa import RSAKey
    >>> rsa_key = RSAKey().load('key.pem')
    >>> rsa_key.has_private_key()
    True

If you have a PEM encoded X.509 certificate you may want to grab the public
RSA key from you could do like this::

    >>> from cryptojwt.jwk.rsa import import_rsa_key_from_cert_file
    >>> from cryptojwt.jwk.rsa import RSAKey
    >>> _key = import_rsa_key_from_cert_file('cert.pem')
    >>> rsa_key = RSAKey(pub_key=_key)
    >>> rsa_key.has_private_key()
    False
    >>> rsa_key.public_key()
    <cryptography.hazmat.backends.openssl.rsa._RSAPublicKey object at 0x1036b1f60>

If you are dealing with Elliptic Curve keys the equivalent would be::

    >>> from cryptojwt.jwk.ec import new_ec_key
    >>> ec_key = new_ec_key('P-256')
    >>> type(ec_key)
    <class 'cryptojwt.jwk.ec.ECKey'>
    >>> ec_key.has_private_key()
    True

and::

    >>> from cryptojwt.jwk.ec import ECKey
    >>> ec_key = ECKey().load('ec-keypair.pem')
    >>> ec_key.has_private_key()
    True

Exporting keys
..............

When it comes to exporting keys, a :py:class:`cryptojwt.jwk.JWK` instance
only knows how to serialize into the format described in JWK_.

    >>> from cryptojwt.jwk.rsa import new_rsa_key
    >>> rsa_key = new_rsa_key()
    >>> rsa_key.serialize()
    {
      'kty': 'RSA',
      'kid': 'NXhZYllJOXdLSW50aUVkcGY4XzZrSVF5blI5aEYxeEJDdFZLV2tHZDlFUQ',
      'n':
        'xRgpX7q-kvQ02EhkHi63TQBR0RMcGCnxCugxtcPmaIX8brilwbkwQyZraEHzWzj-g
         aQyro_dWR7QqbhgiQ6U9Hj3x6HINJuw7LbqR_GE4TvTu3rJXPh3MqTs7yK6GcgKso
         Tv8wQy6Pwl7gjrQRk37zfIHWLkxU-crz2dd1QdSmStlxRjbczik66llF5ENXE3wVz
         raPAdjIv1Y4n5dT3kw7QerVv2Dntn5TJ_8QSkmDJW-FA2TQbKBnOd_OgYeKZnGx5c
         nguWa23uQZTxfGnE7IXA2XYpZhHIgAGMXQ0SaR07MwIZDmreI_Mxypg2ES7XT42qh
         nxXUiGm9fA8nhHjwQ',
      'e': 'AQAB'
    }


What you get when doing it like above is a representation of the public key.
You can also get the values for the private key like this::

    >>> from cryptojwt.jwk.rsa import new_rsa_key
    >>> rsa_key = new_rsa_key()
    >>> rsa_key.serialize(private=True)
    {
      'kty': 'RSA',
      'kid': 'NXhZYllJOXdLSW50aUVkcGY4XzZrSVF5blI5aEYxeEJDdFZLV2tHZDlFUQ',
      'n':
        'xRgpX7q-kvQ02EhkHi63TQBR0RMcGCnxCugxtcPmaIX8brilwbkwQyZraEHzWzj-
         gaQyro_dWR7QqbhgiQ6U9Hj3x6HINJuw7LbqR_GE4TvTu3rJXPh3MqTs7yK6GcgK
         soTv8wQy6Pwl7gjrQRk37zfIHWLkxU-crz2dd1QdSmStlxRjbczik66llF5ENXE3
         wVzraPAdjIv1Y4n5dT3kw7QerVv2Dntn5TJ_8QSkmDJW-FA2TQbKBnOd_OgYeKZn
         Gx5cnguWa23uQZTxfGnE7IXA2XYpZhHIgAGMXQ0SaR07MwIZDmreI_Mxypg2ES7X
         T42qhnxXUiGm9fA8nhHjwQ',
      'e': 'AQAB',
      'd':
        's-2jz73WvqdsGsqzg45YTlZtWrXcXv7jC3b_8pTdoiw3UAkHYXwjYBoR0cLrXCsC
         xO1WS2AQzYxBJ7-neVezih9o7Hl4IPbFJMSzymvlSA1q9OtaKqK1hqljl8gXJvQl
         N-X-e9coduPB6LWBtxNDqgI9kP44JRzRyHUybL6AYuk970_RoqxH2nr8FqMZbNWl
         Vk2X-v06EcO4E_ROSl8vqpb811UidXIvWAJw36LAUw0BTpdvpejSVM1B7PZWbzD9
         1T4vwJYOAVdwWxpmA5HEXRbpNJLnMJus7iq7EVyG2ZbA4TXT-EIoASKMyxJtAuKM
         Dk6cSISWay6LwjdBgVMAAQ',
      'p':
        '588dwE505-i7wL5mWkhH19xS1RzKahFhA66ZVmPjBaA88TBlaZxsdqEADwqXoMq_
         XIUh-P5Tc-ueiCw5rUVNTMb45HWr5fnQXtnJt4yMukNpERABIcWvZWLQg_ONW4iA
         Kid9MLg5EYd2VkAAwXwzzdD1hiYEcxMwQVQ3nLmQ8AE',
      'q':
        '2amgmjQD5Jx7kAR-9oLFjnuvgbUMBOUieQKUCpeJu8q00S7kHb2Hy6ZsanJ--Biu
         1XKz1lxelpN2upsjiKU7f08PB_IPCenBZIU3YwozZd15wCoSyKtffgqk5RXeyi3I
         1ULKXHxr3L7g-7Yi_APgtInQncNnm0Q_t7A_c-P888E'
    }

And you can of course create a key from a JWK representation::

    >>> from cryptojwt.jwk.rsa import new_rsa_key
    >>> from cryptojwt.jwk.jwk import key_from_jwk_dict
    >>> rsa_key = new_rsa_key()
    >>> jwk = rsa_key.serialize(private=True)
    >>> _key = key_from_jwk_dict(jwk)
    >>> type(_key)
    <class 'cryptojwt.jwk.rsa.RSAKey'>
    >>> _key.has_private_key()
    True



Key bundle
----------

As mentioned above a key bundle is used to manage keys that have a common
origin.

You can initiate a key bundle in several ways. You can use all the
import variants we described above and then add the resulting key to a key
bundle::

    >>> from cryptojwt.jwk.ec import ECKey
    >>> from cryptojwt.key_bundle import KeyBundle
    >>> ec_key = ECKey().load('ec-keypair.pem')
    >>> kb = KeyBundle()
    >>> kb.append(ec_key)
    >>> len(kb)
    1
    >>> rsa_key = new_rsa_key()
    >>> kb.append(rsa_key)
    >>> len(kb)
    2
    >>> kb.jwks()
    {"keys":
      [
        {
          "kty": "EC",
          "crv": "B-571",
          "x":
            "AhnRFzRjeOyo-qqm1HPe2JIC69McD29SKAzaBSMzpiRMqh8_tFGWzh2q3pozv_V
             4tsWDgl2H0NdLjNTUQV6JlBn52tJBgCYC",
          "y":
            "LU86ZrZlKMvposLYkgaJrtwklHumK1b1m_joq5r8NsdwQkyVtl44cucurQz1UUc
             oYNJ4ecJv1MWb-I6OwGbiVfM7WSsAhAA",
        },
        {
          "kty": "RSA",
          "kid":
            "cjBobXZ6UHVVdkFmaEJIQXNLbklNNjBMbHo5cWM4U2VjOFRrRkM4RzNaZw",
          "e": "AQAB",
          "n":
            "pyyKp3Fv5ZmyHInUjdEskmI5A-4R19gmzy8SL5waAPd7DMtndQoa-MyMPVP1je
             BPdM_WP17bm1IKt3AWIDpXPq2g0KQxiUU6X9hP738CZaqSmff_hiiT0I3VzsUT
             1SHhdIAeFIeUeuH8RusWo1NnxT7iXRFHbXsG2EOnxr-xB9FZUgMenU4dBIHh2h
             CUW6EZBsYBWuSTyMwRrNp_ZkrH5VJZSCke7bMvlyLlgMFOoDqhuibxEdRmVJAL
             7KkfKQjC0OAd6BXrTk1es590MtMsAIIdKXbgcvxZKeGYSjJ8p8HXLelz50uBhh
             eJdbUmx7MDWCTgouTGzxDaJuCbAR5wMQ",
        }
      ]
    }

**Note** that this will get you a JWKS representing the public keys.

As an example of the special functionality of
:py:class:`cryptojwt.key_bundle.KeyBundle` assume you have imported a file
containing a JWKS with one key into a key bundle and then some time later
another key is added to the file.

First import the file with one key::

    >>> from cryptojwt.key_bundle import KeyBundle
    >>> kb = KeyBundle(source="file://{}".format(fname), fileformat='jwks')
    >>> len(kb)
    1

Now if we add one key to the file and then some time later we ask for the
keys in the key bundle::

    >>> _keys = kb.keys()
    >>> len(_keys)
    2

It turns out the key bundle now contains 2 keys; both the keys that are in the
file.

If the change is that one key is removed then something else happens.
Assume we add one key and remove one of the keys that was there before.
The file now contains 2 keys, and you might expect the key bundle to do the
same::

    >>> _keys = kb.keys()
    >>> len(_keys)
    3

What ???
The key that was removed has not disappeared from the key bundle, but it is
marked as *inactive*. Which means that it should not be used for signing and
encryption but can be used for decryption and signature verification. ::

    >>> len(kb.get('rsa'))
    1
    >>> len(kb.get('rsa', only_active=False))
    2


Key Jar
-------

A key jar keeps keys sorted by owner/issuer. The keys in a key jar are all
part of key bundles.

Creating a key jar with your own newly minted keys you would do:

    >>> from cryptojwt.key_jar import build_keyjar
    >>> key_specs = [
        {"type": "RSA", "use": ["enc", "sig"]},
        {"type": "EC", "crv": "P-256", "use": ["sig"]},
    ]
    >>> key_jar = build_keyjar(key_specs)
    >>> len(key_jar.get_issuer_keys(''))
    3

**Note** that the default issuer ID is the empty string ''.
**Note** also that different RSA keys are minted for signing and for encryption.

You can also use :py:func:`cryptojwt.keyjar.init_key_jar` which will
load keys from disk if they are there and if not mint new.::

    >>> from cryptojwt.key_jar import build_keyjar
    >>> import os
    >>> key_specs = [
        {"type": "RSA", "use": ["enc", "sig"]},
        {"type": "EC", "crv": "P-256", "use": ["sig"]},
    ]
    >>> key_jar = init_key_jar(key_defs=key_specs,
                               private_path='private.jwks')
    >>> len(key_jar.get_issuer_keys(''))
    3
    >>> os.path.isfile('private.jwks')
    True


To import a JWKS you could do it by first creating a key bundle::

    >>> from cryptojwt.key_bundle import KeyBundle
    >>> from cryptojwt.key_jar import KeyJar
    >>> JWKS = {
      "keys": [
        {
           "kty": "RSA",
           "e": "AQAB",
           "kid": "abc",
           "n":
             "wf-wiusGhA-gleZYQAOPQlNUIucPiqXdPVyieDqQbXXOPBe3nuggtVzeq7
              pVFH1dZz4dY2Q2LA5DaegvP8kRvoSB_87ds3dy3Rfym_GUSc5B0l1TgEob
              cyaep8jguRoHto6GWHfCfKqoUYZq4N8vh4LLMQwLR6zi6Jtu82nB5k8"
        }
      ]}
    >>> kb = KeyBundle(JWKS)
    >>> key_jar = KeyJar()
    >>> key_jar.add_kb('', kb)

The last line can also be expressed as::

    >>> keyjar[''] = kb

**Note** both variants add a key bundle to the list of key bundles that
belong to '', it does not overwrite anything that was already there.

Adding a JWKS is such a common thing that there is a simpler way to do it::

    >>> from cryptojwt.key_jar import KeyJar
    >>> JWKS = {
      "keys": [
        {
           "kty": "RSA",
           "e": "AQAB",
           "kid": "abc",
           "n":
             "wf-wiusGhA-gleZYQAOPQlNUIucPiqXdPVyieDqQbXXOPBe3nuggtVzeq7
              pVFH1dZz4dY2Q2LA5DaegvP8kRvoSB_87ds3dy3Rfym_GUSc5B0l1TgEob
              cyaep8jguRoHto6GWHfCfKqoUYZq4N8vh4LLMQwLR6zi6Jtu82nB5k8"
        }
      ]}
    >>> key_jar = KeyJar()
    >>> key_jar.import_jwks(JWKS)

The end result is the same as when you first created a key bundle and then
added it to the key jar.

When dealing with signed and/or encrypted JSON Web Tokens
:py:class:`cryptojwt.key_jar.KeyJar` has these nice methods.

    get_jwt_verify_keys
        :py:func:`cryptojwt.key_jar.KeyJar.get_jwt_verify_keys` takes a
        signed JWT as input and returns a set of keys that
        can be used to verify the signature. The set you get back is a best
        estimate and might not contain **the** key. How good the estimate is
        depends on the information present in the JWS.

    get_jwt_decrypt_keys
        :py:func:`cryptojwt.key_jar.KeyJar.get_jwt_decrypt_keys` does the
        same thing but returns keys that can be used to decrypt a message.


.. _cryptography: https://cryptography.io/en/latest/
.. _JWK: https://tools.ietf.org/html/rfc7517
.. _JWKS: https://tools.ietf.org/html/rfc7517#section-5
.. _JWS: https://tools.ietf.org/html/rfc7515
