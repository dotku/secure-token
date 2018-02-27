# `secure-token`

[![Build Status](https://travis-ci.org/emilbayes/secure-token.svg?branch=master)](https://travis-ci.org/emilbayes/secure-token)

> Simple, secure tokens for authentication, access keys, sessions etc.

## Usage

Below is an example using `secure-token` stored in cookies. Note that you use
`secureToken.hash` to store and verify a token:

```js
var secureToken = require('secure-token')
var db = new Map() // Use map as database for simplicity

function login (req, res) {
  // Do authentication
  // ...
  // If success issue a session token
  var sessionToken = secureToken.create()

  // Here we use the 'session'
  db.set(secureToken.hash(sessionToken, 'session'), true)

  res.writeHead(204, {
    'Set-Cookie': [
      `sessionToken=${secureToken.toString('base64')}`,
      'HttpOnly',
      'Secure'
    ].join(';')
  })
  res.end()
}

function secretPage (req, res) {
  // Get req.sessionToken somehow
  var sessionToken = Buffer.from(req.sessionToken, 'base64')

  var hash = secureToken.hash(sessionToken, 'session')

  if (!db.get(hash)) {
    res.writeHead(400)
    return res.end()
  }

  res.writeHead(200)
  return res.end('Yay!')
}
```

## API

### `var tokenBuf = secureToken.create([size])`

Create a new token from your OS Cryptographically Secure Pseudorandom Number
Generator (CSPRNG), making the token unpredictable and return as a `Buffer`.
`size` defaults to 18, giving a security level of more than 128 bits, while
avoiding any padding when Base 64 encoded.

### `var hashBuf = secureToken.hash(tokenBuf, [namespace])`

Hash a token for long-term storage, taking `Buffer` `tokenBuf` and an optional
`namespace` which can be either a string or `Buffer`. You can use `namespace` to
partition your tokens for different use-cases, invalidating tokens which are
used for the wrong purpose, while keeping the information hidden in storage.
`namespace` does not add any significant security and is simply so that
different tokens are not used in the wrong context.

`tokenBuf` should be a token generated by `secureToken.create` and namespace
can be a `Buffer` or `String`.

The reason it is important to obscure the token is that it is password
equivalent, meaning having access to a valid token is the same as having gone
through an authentication process, eg. typing a password. You do not want anyone
with access to your tokens to be able to impersonate a user.

Using the default token size it should take well over 2^64 guesses to find two
tokens that yield the same hash value due to the birthday paradox.


## Install

```sh
npm install secure-token
```

## License

[ISC](LICENSE.md)
