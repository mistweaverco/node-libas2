# libas2

> [!IMPORTANT]  
> This project is a hard-fork of the original [node-libas2](https://github.com/aaronhuggins/node-libas2)
> It has some modifications to the original code, but isn't a complete rewrite.
> 
> The original project is no longer maintained,
> and this fork aims to provide a more stable and
> up-to-date version of the library.

A pure Javascript/Typescript implementation of the AS2 protocol.

This project assumes that it fully implements Applicability Statement 2 (AS2) version 1.0 per RFC 4130.

Best effort has been made to achieve tests which cover the different aspects of the RFC,
but **this isn't a certified library**.

The project does not have access to Drummond certification,
which is considered to be the standards body in certifying compatibility with AS2,
so good sense should be used when using this library as the AS2 layer in an application.

## Usage

Install it from the [npm repository](https://www.npmjs.com/package/@mistweaverco/libas2):

```sh
npm install --save @mistweaverco/libas2
```

Then import it in your project:

```typescript
import { AS2Composer } from '@mistweaverco/libas2';
```

## Features

- Compose AS2 messages to send, including signing and encrypting
- Parse received AS2 messages, including decrypting and verifying
- Generate and consume Message Disposition Notifications; includes convenience methods for incoming and outgoing dispositions
- Complete support for AS2 protocol 1.0 [per RFC 4130](https://tools.ietf.org/html/rfc4130)
- Rich cryptography support based on [PKIjs](https://github.com/PeculiarVentures/PKI.js)
- Rich MIME support based on [Nodemailer](https://github.com/nodemailer/nodemailer) and [Emailjs](https://github.com/emailjs/emailjs-mime-parser)

## Examples

```typescript

import {
  AS2Constants,
  AS2Composer,
  type AS2ComposerOptions,
  request
} from '@mistweaverco/libas2';
import { configuration } from './config';
import fs from 'fs';

const LIBAS2_CERT = fs.readFileSync('certificate.crt', 'utf8')
const LIBAS2_KEY = fs.readFileSync('certificate.key', 'utf8');
const LIBAS2_CERT_AMAZON = fs.readFileSync('certificate-amazon.crt', 'utf8')

const options: AS2ComposerOptions = {
  message: {
    filename: 'msg.edifact',
    contentType: 'application/EDIFACT',
    content: ediMessage,
  },
  agreement: {
    host: {
      name: `MWCO ${configuration.region}`,
      id: 'MWCEDIAS24007',
      url: 'http://35.207.79.189:8085',
      certificate: LIBAS2_CERT,
      privateKey: LIBAS2_KEY,
      decrypt: false,
      sign: true,
      mdn: {
        async: false,
        signing: AS2Constants.SIGNING.SHA256
      }
    },
    partner: {
      name: 'Amazon',
      id: 'SE1Y1CG1Q1OF1QT',
      url: 'http://as2-eu.amazonsedi.com/a331d41b-e681-461c-b40f-db6b143b213b',
      file: 'EDIFACT',
      certificate: LIBAS2_CERT_AMAZON,
      encrypt: AS2Constants.ENCRYPTION.AES128_CBC,
      oaepHashAlgorithm: AS2Constants.OAEP_HASH_ALGORITHMS.SHA1,
      verify: true
    }
  },
}

const composer = new AS2Composer({
  message: options.message,
  agreement: options.agreement,
})

const result = await request(await composer.toRequestOptions())
const mdn = await result.mime()
const message = await mdn.verify({ cert: LIBAS2_CERT_AMAZON })
console.log('MDN received:', message);
```
