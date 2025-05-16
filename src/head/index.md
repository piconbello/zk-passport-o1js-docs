# Understanding Key Data Structures in eMRTDs

This document outlines the core data elements and structures involved in the security mechanisms of electronic Machine Readable Travel Documents (eMRTDs), such as ePassports. It details the journey from the physically printed data to the digitally signed components on the chip.

## 1. Machine Readable Zone (MRZ)

- **Definition:** The MRZ is the set of lines printed in a machine-readable font on the data page of a travel document. It's designed for optical character recognition (OCR) by document readers.
- **Formats (e.g., TD3):**
  - ICAO specifies different MRZ formats for various document types. A common one for passports is **`TD3`**.
  - A `TD3` MRZ consists of two lines, each 44 characters long, totaling 88 characters. These characters encode key biographic and document information.
  - Other formats like `TD1` (e.g., ID cards, 3 lines of 30 characters) and `TD2` (other card-format documents, 2 lines of 36 characters) also exist.
- **Structure:** Composed of uppercase alphanumeric characters (A-Z, 0-9) and the filler character `<`. The content and position of data elements within these lines are strictly defined by ICAO Doc 9303.
- **Purpose:** Provides a standardized way for automated systems to quickly capture essential holder and document information.

## 2. Data Group 1 (DG1)

- **Definition:** `DG1` is an elementary file stored on the eMRTD's integrated circuit (chip). It is the electronic representation of the data contained in the physical MRZ.
- **Relationship to MRZ:** The content of `DG1` directly mirrors the data from the MRZ. For a document with a `TD3` MRZ, `DG1` will store those 88 characters.
- **Structure:**
  - The core data within `DG1` is typically an ASN.1 `IA5String` containing the concatenated lines of the MRZ.
  - The actual `EF.DG1` file on the chip is ASN.1 encoded. This means the 88-character string (for `TD3`) is wrapped with an ASN.1 tag (identifying it as `DG1`, typically `61` hex) and a length field. For an 88-character `TD3` MRZ, this commonly results in `DG1` being 93 bytes long on the chip (e.g., `61 5B 5F 1F 58 <88 bytes of MRZ data>`).
- **Purpose:** Allows electronic access to the MRZ information, forming a foundational data element for security checks performed by inspection systems.

## 3. LDS Security Object (`SOd` or `LDSSecurityObject`)

- **Definition:** The `SOd` (Security Object document), more formally referred to as `LDSSecurityObject` in some ASN.1 definitions, is a critical data structure stored on the eMRTD chip. Its primary function is to ensure the integrity of the various Data Groups (like `DG1`, `DG2` for the photo, etc.).
- **Relationship to DG1 (and other DGs):** The `SOd` contains cryptographic hashes (digital fingerprints) of specified Data Groups. For instance, it will include a hash calculated over the entire content of `DG1`.
- **Structure (Conceptual ASN.1):** The `LDSSecurityObject` is an ASN.1 `SEQUENCE` typically containing:
  1.  `version`: An integer indicating the version of the `SOd` structure (e.g., 0).
  2.  `hashAlgorithm`: An `AlgorithmIdentifier` (an OID) specifying the cryptographic hash algorithm (e.g., SHA-256, SHA-384) used to compute the digests of the Data Groups.
  3.  `dataGroupHashValues`: A `SEQUENCE OF` or `SET OF` structures, often named `DataGroupHash` or `DatagroupDigest`. Each of these structures within the list contains:
      - `dataGroupNumber`: An integer identifying the Data Group (e.g., 1 for `DG1`, 2 for `DG2`).
      - `dataGroupHashValue`: An `OCTET STRING` representing the cryptographic hash (digest) of the content of the specified Data Group.
- **Encoding:** The entire `LDSSecurityObject` structure is DER (Distinguished Encoding Rules) encoded before being processed further or stored.
- **Purpose:** To enable verification that the Data Groups stored on the chip have not been altered since the `SOd` was created and signed.
- **Important remark:** DG1 is always at the first location, the prefix length is deterministic. The total size is dependent on the digest algorithm size and how many data groups exist.

## 4. SignedAttributes

- **Definition:** `SignedAttributes` is a specific ASN.1 structure defined within the Cryptographic Message Syntax (CMS, RFC 5652). In the context of eMRTDs, it's a collection of attributes that are DER-encoded and then become the _actual data_ that is digitally signed by the issuing authority's Document Signer Certificate (DSC).
- **Relationship to `SOd`:** The `SignedAttributes` structure _includes_ a cryptographic hash of the entire DER-encoded `SOd`.
- **Structure (Conceptual ASN.1, within CMS `SignerInfo`):**
  - `SignedAttributes ::= SET SIZE (1..MAX) OF Attribute`
  - Each `Attribute` is a `SEQUENCE` of:
    - `attrType OBJECT IDENTIFIER`: Identifies the type of attribute.
    - `attrValues SET OF AttributeValue`: Contains the value(s) for that attribute.
  - For eMRTD `SOd` signing, critical `SignedAttributes` include:
    - **`contentType` (OID: `1.2.840.113549.1.9.3`):** The `AttributeValue` for this is an OID. For an eMRTD `SOd`, this value is `id-icao-ldsSecurityObject` (OID: `0.4.0.127.0.7.2.2.1`), indicating that the signed content's "type" is an ICAO `SOd`.
    - **`messageDigest` (OID: `1.2.840.113549.1.9.4`):** The `AttributeValue` for this is an `OCTET STRING` containing the cryptographic hash of the entire DER-encoded `SOd`.
    - **(Often) `aaSigningCertificateV2` (OID: `1.2.840.113549.1.9.16.2.47`):** An ICAO-defined attribute whose value (an `ESSCertIDv2` structure) contains a hash of the Document Signer Certificate, linking the signature back to the specific certificate.
- **Encoding:** The collection of these attributes is itself DER-encoded to form the octet string that is input to the digital signature algorithm.
- **Purpose:**
  - To ensure that the digital signature applies to a specific, well-defined set of information, including the hash of the `SOd` and the type of content.
  - This prevents replay attacks or misinterpretation of the signature's scope. The signature on `SignedAttributes` cryptographically binds these attributes to the `SOd`.
- **Important remark:** The size of the `SignedAttributes` is determined by the digest algorithm size and the algorithm's encoded `OID` length.

## Summary of Relationships

The data elements are processed and related in a layered manner for security:

1.  **MRZ (e.g., TD3):** Printed on the physical document.
2.  **DG1:** Electronic copy of the MRZ data, stored on the chip.
3.  **`SOd` (`LDSSecurityObject`):** Contains a list of Data Group numbers and their corresponding cryptographic hashes (e.g., hash of DG1, hash of DG2).
4.  **`SignedAttributes`:** Contains a hash of the _entire `SOd`_ (after DER encoding), along with other metadata like `contentType`.
