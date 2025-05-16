# Glossary

- **NFC**: Near Field Communication
- **OCR**: Optical Character Recognition
- **ASN1**: Abstract Syntax Notation One.
- **MRZ**: Machine Readable Zone (read data with OCR)  
  TD3 example:
  ```
  P<GBRBAGGINS<<FRODO<<<<<<<<<<<<<<<<<<<<<<<<<
  P231458901GBR6709224M2209151ZE184226B<<<<<18
  ```
- **eMRTD:** Electronic Machine Readable Travel Document (read data with NFC)
  - Contains several data groups each encoded in ASN1 format.
- **x509**: A standard of certificates for digital signatures and more, uses ASN1 format.
- **Masterlist**: List of trusted root x509 certificates, issued to countries for eMRTD.
- **ICAO**: International Civil Aviation Organization, distributes masterlists for eMRTD.
- **DG1**: Data Group 1 of eMRTD data, exactly the same as MRZ information.
- **CMS**: Cryptographic Message Syntax, used to represent a payload, signature, and an x509 certificate in ASN1 format.
- **SOD**: Document Security Object of eMRTD data, contains a CMS formatted data (hashes of data groups enveloped) with an x509 certificate.
- **o1js:** TypeScript framework for zk-SNARKs and zkApps
- **zkProgram:** Creates off-chain zero-knowledge proofs with o1js.
- **npm**: Node Package Manager, a registry for node.js and javascript packages.
- **npm**: Node Package Manager, a registry for node.js and javascript packages.

```

```
