---
title: Security Aspects of Key Identifiers on COSE/JOSE

abbrev: Security Aspects of Key Identifiers on COSE/JOSE
docname: draft-tschofenig-jose-key-identifier-security-latest
category: std

ipr: trust200902
area: Security
workgroup: JOSE
keyword: Internet-Draft

stand_alone: yes
pi:
  rfcedstyle: yes
  toc: yes
  tocindent: yes
  sortrefs: yes
  symrefs: yes
  strict: yes
  comments: yes
  inline: yes
  text-list-symbols: -o*+
  docmapping: yes
author:
 -
      ins: H. Tschofenig
      name: Hannes Tschofenig
      email: hannes.tschofenig@gmx.net
      org: University of Applied Sciences Bonn-Rhein-Sieg
      abbrev: H-BRS

 -
      ins: L. Hazlewood
      name: Les Hazlewood
      email:  lhazlewood@gmail.com
      org:

 -
      ins: Y. Sheffer
      name: Yaron Sheffer
      email:  yaronf.ietf@gmail.com
      org: Intuit

normative:
  RFC2119:
  RFC7515:
  RFC7516:
  RFC8725:
  RFC9052:
informative:

--- abstract

This document provides guidance for improving the security of JSON Object Signing and Encryption (JOSE) and CBOR Object Signing and Encryption (COSE) implementations. It emphasizes the importance of handling key identification within the header to simplify security processing and reduce risks. Recommendations are given to ensure better interoperability and security for protocol designers and implementers

--- middle

#  Introduction

JSON Object Signing and Encryption (JOSE) was originally designed to provide a security wrapper for access tokens used in the OAuth protocol, focusing particularly on digital signatures. However, its utility as a standard for describing security-related metadata was quickly recognized. Today, JOSE is widely adopted and its functionality spans across various specifications (such as {{RFC7515}} for JSON Web Signature and {{RFC7516}} for JSON Web Encryption). {{RFC8725}} was published to describe JSON Web Token (JWT) best current practices to react to problems with the use of JOSE in context of JWTs.

With the development of CBOR {{!RFC8949}}, a binary encoding format was introduced to address use cases where JSON was too verbose. A security wrapper utilizing CBOR-based encoding was required, leading to the standardization of CBOR Object Signing and Encryption (COSE), further refined by {{!RFC9052}} and {{!RFC9053}}.

The JOSE and COSE specifications have intentionally been kept in sync because modern protocols and payloads are frequently described in the Concise Data Definition Language (CDDL) and serialized to either JOSE or COSE formats. This convergence makes them attractive to developers working across web and embedded systems. Due to their similar designs, the guidance provided in this document is applicable to both JOSE and COSE.

However, certain practices pose security challenges. This document addresses security challenges and provides guidance for improving JOSE/COSE extensions, emphasizing key identification. Key identification should be handled within the header to maintain a clear separation from the payload, simplifying security processing and reducing risks.

The document is structured as follows: {{kid}} outlines challenges related to key identification. {{guidance}} provides recommendations.

# Terminology and Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in RFC 2119 {{RFC2119}}.

#  Key Identification {#kid}

The security wrappers in JOSE and COSE employ a straightforward design, especially for basic functionalities like digital signatures and MACs aimed at a single recipient.

The security wrapper comprises the following components:

- A header, divided into protected and unprotected parameters.

- The payload, which may be detached and transmitted independently. This payload requires protection and often consists of a JSON-based format (for JOSE) or a CBOR-encoded format (for COSE). Standardized payloads include JSON Web Tokens (JWT) {{!RFC7519}} and CBOR Web Tokens (CWT) {{!RFC8392}}.

- A digital signature, a tag (for MAC), or ciphertext (for encryption).

The header's purpose is to provide instructions for protecting the payload, including:

- Algorithm information used to protect the payload,
- Key identification for verifying the digital signature, MAC, or encryption,
- X.509 certificates and certificate chains,
- Countersignature.

While the layering is straightforward with the header providing instructions for payload protection, certain specifications and applications have begun embedding key identification information within the payload itself, disrupting this clear separation.

Using the 'kid' parameter is the recommended approach for key identification, although {{RFC7515}} does not mandate that key identification values be globally unique (and hence "collision resistant"). If a JOSE- or COSE-protected message is intended for external or third-party recipients:

- The 'kid' parameter MUST contain a globally unique value, or
- Other header parameters, when combined with 'kid', must result in a globally unique value.

For JOSE/COSE-protected messages used within domain-specific contexts like enterprises or specific workloads, uniqueness requirements are relaxed.

The practice of placing key identification information into the payload instead of the JOSE/COSE header forces a parser to postpone security processing until later. The parser must inspect the payload to find the appropriate keying material and subsequently verify it. Since the parser does not know in advance which fields contain key identification, it must expose all information to the application before signature verification or MAC processing. This introduces significant risk, as application developers may make security decisions before completing security processing.

This design is unnecessary because existing header parameters can store this information. If these headers are insufficient, new header parameters can always be defined to convey necessary information. This approach also simplifies libraries, as they do not need to understand payload content to retrieve correct information.

When key identification claims are placed in the payload, they SHOULD also be duplicated in the header, as specified in {{!I-D.ietf-cose-cwt-claims-in-headers}} (for COSE) and {{Section 5.3 of !RFC7519}} (for JOSE). This approach should only be used as a last resort if the previous methods cannot be implemented.

Finally, transitioning seamlessly from a system using digital signatures over payloads to encrypted payloads is challenging when necessary key lookup information is embedded within the encrypted payload. A redesign is therefore necessary.

# Guidance {#guidance}

We RECOMMEND that protocol designers and implementers utilize the
available header parameter for key identification. If the standardized
parameters are insufficient, new header parameters can be defined.
Re-using existing header parameters will improve interoperability
because there are a limited number of cases on how to select a key.

Information required to determine the keying material for cryptographically verifying the protected payload MUST be placed in the header of the JOSE/COSE security wrapper.

#  IANA Considerations

This document does not make requests to IANA.

#  Security Considerations

This specification makes security recommendations for the
JOSE/COSE specification suite. Therefore, it is entirely
about security.

--- back

# Acknowledgments

We would like to thank Casten Bormann for his review comments.
