# couch-box
Asymmetric encrypted CouchDB documents, powered by NaCl's curve25519-xsalsa20-poly1305.

[![Build Status](https://travis-ci.org/jo/couch-box.svg?branch=master)](https://travis-ci.org/jo/couch-box)

couch-box uses [TweetNaCl.js](https://github.com/dchest/tweetnacl-js), a port of
[TweetNaCl](http://tweetnacl.cr.yp.to/) / [NaCl](http://nacl.cr.yp.to/) to
JavaScript for modern browsers and Node.js by Dmitry Chestnykh
([@dchest](https://github.com/dchest)).

The use of the widely ported cryptography library NaCl makes it possible to
implement this encryption schema in other, possibly more secure platforms, for
example with Python and CouchDB.

**:warning: Only to play around! Not yet ready for production use.**

## Installation
couch-box is [hosted on npm](https://www.npmjs.com/package/couch-box).

### Node
Install via `npm install couch-box` 

## Usage
```js
var databaseKeyPair = require('tweetnacl').box.keyPair()
// a keyPair consists of a `publicKey` and a `secretKey` (here base64 encoded):
// {
//   secretKey: 'smsDNnqeT40IfAwDw0+6x5WzDRYFv0492O/JW/s8tT0=',
//   publicKey: 'sAUGULAT5q2g6gzNMuBX1tkY/FsnoiLA/tv2XmmU2Dg='
// }

// create a box
var box = require('tweetnacl')(databaseKeyPair)

var doc = {
  _id: 'mydoc',
  // everything inside `box` gets encrypted
  box: {
    text: 'a secret text'
  },
  // nothing outside `box` will be touched by couch-box
  public: 'some public visible property'
}

// encrypt doc
box(doc)
// doc looks now like this:
// {
//   _id : 'mydoc',
//   public: 'some public visible property',
//   box: {
//     ephemeral : 'PuiUBvQY+7ZFPXXUQ1N2eNE9tgPgIkT1uWj9rpShwXY=',
//     nonce: 'zGDblW4Ov8sMKG3YcV/BISueH+REtDr3',
//     receivers: {
//       '2XiwPX1U6pKPitmhyeubV9g4YYxtIxNfMNE6B5keEmg=': {
//         nonce: 'pSquTTn+/I7REorstK6hSYeKizajtu65',
//         encryptedKey: 'GXEfX7V3IwA0izAAJ3HIRCzxDFIUfxMq82QO49ITwKzbi+S+5TanJ/9ubmxOUyBh'
//       }
//     },
//     cipher: 'D9xRZl+/k0gvdBx33CGKaGfLTH731T6jhkMXfh9GfVxETGmTcpzqSJNQ42GPzsafycpdSd7ZTTWBO2vXu06dCha/X8P8C+F6Po+LeerJhKgG'
//   }
// }

// decrypt doc
box.open(doc)
// doc has been decrypted and receiver stubs have been added
// {
//   _id : 'mydoc',
//   public: 'some public visible property',
//   box: {
//     receivers: {
//       '2XiwPX1U6pKPitmhyeubV9g4YYxtIxNfMNE6B5keEmg=': true
//     },
//     text: 'a secret text'
//   }
// }

// and later...
box.close()
```

## Details
Each document is encrypted with its own key. For each database key which was
given access to the document a permit is included in the document. This empowers
the owner to grant access to other accounts on a per document basis.

Each document has its own key which is used together with [Nacl secret-key
authenticated encryption](http://nacl.cr.yp.to/secretbox.html). The key consists
of 32 random bytes.

In order to create the doc permit we
1. Create an ephemeral key pair
2. Create a nonce
3. Encrypt the document key with nonce, public database key and ephemeral secret key

```json
{
  "_id" : "a069f1041735910cf8f613d20000116b",
  "box": {
    "ephemeral" : "PuiUBvQY+7ZFPXXUQ1N2eNE9tgPgIkT1uWj9rpShwXY=",
    "nonce": "zGDblW4Ov8sMKG3YcV/BISueH+REtDr3",
    "receivers": {
      "2XiwPX1U6pKPitmhyeubV9g4YYxtIxNfMNE6B5keEmg=": {
        "nonce": "pSquTTn+/I7REorstK6hSYeKizajtu65",
        "encryptedKey": "GXEfX7V3IwA0izAAJ3HIRCzxDFIUfxMq82QO49ITwKzbi+S+5TanJ/9ubmxOUyBh"
      }
    },
    "cipher": "D9xRZl+/k0gvdBx33CGKaGfLTH731T6jhkMXfh9GfVxETGmTcpzqSJNQ42GPzsafycpdSd7ZTTWBO2vXu06dCha/X8P8C+F6Po+LeerJhKgG"
  }
}
```

## Testing
```sh
npm test
```
