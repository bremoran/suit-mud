---
title: Strong Assertions of IoT Network Access Requirements
abbrev: SUIT MUD Linkage
updates: draft-ietf-suit-manifest
docname: draft-ietf-suit-mud-08
category: std

ipr: trust200902
area: Security
workgroup: SUIT
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
      ins: B. Moran
      name: Brendan Moran
      organization: Arm Limited
      email: brendan.moran.ietf@gmail.com

 -
      ins: H. Tschofenig
      name: Hannes Tschofenig
      organization: 
      email: hannes.tschofenig@gmx.net

normative:
  RFC8520:
  RFC2119:
  RFC8174:
  I-D.ietf-rats-eat:
  I-D.ietf-suit-manifest:
  I-D.ietf-cose-key-thumbprint:
  RFC8610:
  RFC9334:
informative:
  RFC9190:
  I-D.fossati-tls-attestation:
  FDO:
    author:
      org: FIDO Alliance
    title: FIDO Device Onboard Specification 1.1
    date: April 2022
    target: https://fidoalliance.org/specifications/download-iot-specifications/

--- abstract

The Manufacturer Usage Description (MUD) specification describes the access and network functionality required for a device to properly function. This description has to reflect the software running on the device and its configuration. Because of this, the most appropriate entity for describing device network access requirements is the same as the entity developing the software and its configuration.

A network presented with a MUD file by a device allows detection of misbehavior by the device software and configuration of access control.

This document defines a way to link the Software Updates for Internet of Things (SUIT) manifest to a MUD file offering a stronger binding between the two.

--- middle

# Introduction

A Manufacturer Usage Description (MUD) file describes what sort of network communication behavior a device is designed to have. For example,
a manufacturer may use a MUD file to describe that a device uses HTTP, DNS and NTP communication but no other protocols. The communication patterns are
described in a JSON-based format in the MUD file.

The MUD files do, however, need to be presented by the device to a MUD manager in the operational network where the device is deployed.
Under {{RFC8520}}, devices report a MUD URL to a MUD manager in the operational network. The MUD URL is a URL that can be used by the MUD manager to receive the MUD file from a MUD file server to ultimately obtain the MUD file.

{{arch-mud-fig}} shows the MUD architecture, as defined in RFC 8520.

~~~
    .......................................
    .                      ____________   .           _____________
    .                     |            |  .          |             |
    .                     |    MUD     |-->get URL-->|    MUD      |
    .                     |  Manager   |  .(https)   | File Server |
    .  End system network |____________|<-MUD file<-<|_____________|
    .                             .       .
    .                             .       .
    . ________                _________   .
    .|        |              | router  |  .
    .| Device |--->MUD URL-->|   or    |  .
    .|________|              | switch  |  .
    .                        |_________|  .
    .......................................
~~~
{: #arch-mud-fig title="MUD Architecture per RFC 8520."}

RFC 8520 envisions different approaches for conveying the MUD URL from the device to the operational network. Section 4 of {{?I-D.ietf-opsawg-mud-acceptable-urls}} provides additional description of the MUD URLs sources, which include:

- DHCP,
- IEEE 802.1AB Link Layer Discovery Protocol (LLDP), and
- IEEE 802.1X whereby the URL to the MUD file would be contained in the certificate used in an EAP method.

The MUD manager must trust the MUD file server from which the MUD file is fetched to return the most up-to-date MUD file.
It must also trust the device to report the correct MUD URL. In case of DHCP and LLDP the URL is unprotected and not bound
to the device itself.

When the MUD URL is included in a certificate then it is authenticated and integrity protected. However, the certificate only proves possession of a private key and endorsements by the certificate issuer. This does not prove what software is in use, nor does it prove that the MUD file is the correct file for the deployed software: instead, the responsibility falls on the certificate issuer to identify the MUD URL correctly and to supply a MUD Signer correctly.
There is a need to bind the entity that creates the software and configuration to the MUD file. The developer is in the best position to describe
the communication requirements of the software it developed and configured for a device.

This specification defines an extension to the Software Updates for Internet of Things (SUIT) manifest {{I-D.ietf-suit-manifest}}
to include a MUD URL. A SUIT manifest is
   a bundle of metadata about code/data for an IoT device, where to find the code/data, the
   devices to which it applies, and cryptographic information protecting
   the manifest.

 When combining a MUD URL with a manifest used for software/firmware updates then a network operator can gain
more confidence in the description of the communication requirements for a device to properly function.

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all
capitals, as shown here.

This document re-uses the terms defined in {{RFC9334}} related to
remote attestation.  Readers of this document are assumed to be
familiar with the following terms: Evidence, Claim, Attester, Verifier,
and Relying Party (RP).

This document also uses terms defined in {{RFC8520}}, such as MUD,
MUD file, MUD manager, MUD URL, etc.

# Workflow

{{arch-mud-new-fig}} shows the architectural extensions introduced by combining
SUIT and MUD. The key elements are that the developer, who produces the
firmware is also generating a manifest and the MUD file. Information
about the MUD file is embedded into the SUIT manifest and provided to the
device via firmware update mechanism. Once this information is available
on the device it can be presented during device onboarding, during
network access authentication, or as part of other interactions that
involve the conveyance of Evidence to the operational network. After
retrieving the manifest, the MUD file can be obtained as well.

~~~
                        ____________
                       |            |
                       |  Manifest  |
                       | Repository |
                       |____________|
                  get URL ^      | SUIT manifest
 .........................|......|..........
 .                      __|______v__       .       _____________
 .                     |            |      .      |             |
 .                     |    MUD     |-->get URL-->|    MUD      |
 .                     |  Manager   |  .(https)   | File Server |
 .  End system network |____________|<-MUD file<-<|             |
 .                             ^       +Signature |_____________|
 .                             .           .
 .                             .           .
 .                             .           .
 . ________                _____________   .
 .|        | Attestation  | NAS, AAA or |  .
 .| Device |-->Evidence-->| Onboarding  |  .
 .|________| (+ Manifest  | Server      |  .
 .     ^      Claim)      |_____________|  .
 ......*....................................
       *                                         //-\\
       *                                          \-/
       *                        SUIT Manifest      |
       +************************(+ MUD URL)    ----*-----
                                Firmware          / \
                                                  /  \
                                               Developer
~~~
{: #arch-mud-new-fig title="SUIT-MUD Architecture."}

The intended workflow is as follows, and assumes an attestation mechanism between the device and the MUD Manager:

*  At the time of onboarding, devices report their manifest in use to the MUD Manager via some form of attestation Evidence and a conveyance protocol. The device thereby acts as an Attester. The normative specification of these mechanisms is out of scope for this document.

  -  An example of an Evidence format is the Entity Attestation Token (EAT) {{I-D.ietf-rats-eat}}, which offers a rich set of claims. This specification assumes that Evidence includes a link to the SUIT manifest via the "manifests" claim (see Section 4.2.15 of {{I-D.ietf-rats-eat}}) or that the manifest itself is embedded in the Evidence. This Evidence is conveyed to the operational network via some protocol, such as network access authentication protocol (for example using the EAP-TLS 1.3 method {{RFC9190}} utilizing the attestation extensions {{I-D.fossati-tls-attestation}}) or an onboarding protocol like FIDO Device Onboard (FDO) {{FDO}} or Bootstrapping Remote Secure Key Infrastructure (BRSKI) {{?RFC8995}}.

  -  The MUD Manager, acting as a Relying Party, relays the Evidence to the Verifier and receives an Attestation Result in response. This allows the MUD Manager to check that the device is operating with the expected version of software and configuration.

  -  Since a URL to the manifest is contained in the Evidence, the MUD Manager can look up the corresponding manifest.

* The MUD Manager acquires the MUD file from the MUD URL found in the SUIT manifest. The SUIT manifest contains the MUD URL and not the MUD file primarily to due the size of the MUD file. This also allows the MUD file to be updated rapidly in response to evolving threats.
* The MUD Manager verifies the MUD file signature using the Subject Key Identifier (SKI) provided in the SUIT manifest.
* Then, the MUD Manager can apply any appropriate policy as described by the MUD file.

Each time a device is updated, rebooted, or otherwise substantially changed, it will execute the remote attestation procedures again.

# Operational Considerations

This specification assumes that the software/firmware author provides a MUD file that describes the behavior of the software running on a device.

## Pros

The approach described in this document has several advantages over the RFC 8520 MUD URL reporting mechanisms:

* The MUD URL is tightly coupled to device software/firmware version.
* The device does not report the MUD URL, so the device cannot tamper with the MUD URL.
* The author explicitly authorizes a key to sign MUD files, providing a tight coupling between the party that knows device behavior best (the author of the software/firmware) and the party that declares device behavior (MUD file signer).
* Network operators do not need to know, a priori, which MUD URL to use for each device; this can be harvested from the device's manifest and only replaced if necessary.
* A network operator can still replace a MUD URL in a SUIT manifest:

  * By providing a SUIT manifest that overrides the MUD URL.
  * By replacing the MUD URL in their network infrastructure.

* Devices can be quarantined if the Attestation Result indicates that an out-dated or compromised software/firmware version has been used.
* Devices cannot lie about which MUD URL to use.

## Cons

This mechanism relies on the use of SUIT manifests to encode the MUD URL. Conceptually, the MUD file is similar to a Software Bill of Material (SBOM) but focuses on the external visible communication behavior, which is essential for network operators, rather than describing the software libraries contained within the device itself. The SUIT manifest must then be conveyed to the network during onboarding or during the network access authentication step. Attestation Evidence is used to convey the SUIT manifest. 

# Extensions to SUIT {#suit-extension}

To enable strong assertions about the network access requirements that a device should have for a particular software/configuration pair a MUD URL is added to the SUIT manifest along with a subject key identifier (ski). Note that the subject key identifier refers to a more generic version of SubjectPublicKeyInfo defined in {{?RFC5280}}, which refers to an X.509-based ski.
The subject key identifier MUST be generated according to the process defined in {{I-D.ietf-cose-key-thumbprint}} and the SUIT_Digest structure MUST be populated with the selected hash algorithm and obtained fingerprint.
The subject key identifier corresponds to the key used in the MUD signature file described in Section 13.2 of {{RFC8520}}.

Note: A key need not be in COSE Key format to create a COSE Key Thumbprint of it.

The following Concise Data Definition Language (CDDL) {{RFC8610}} describes the extension to the SUIT_Manifest structure:

The extension to the SUIT_Manifest is described here:

~~~CDDL
$$unseverable-manifest-member-extensions //= (
  suit-manifest-mud => bstr .cbor SUIT_MUD_container
)
~~~

The SUIT_MUD_container structure is defined as follows:

~~~CDDL
SUIT_MUD_container = {
    suit-mud-url => #6.32(tstr),
    suit-mud-ski => SUIT_Digest,
}
~~~

# Security Considerations

This specification links MUD files to SUIT manifests for improving security protection and ease of use. By including MUD URLs in SUIT manifests an extra layer of protection has been created and synchronization risks can be minimized.

Used in this way, the MUD manager presents an additional layer of security on networks where they are enabled. The MUD manager configures the L2/L3 infrastructure of a Local Area Network to apply restrictive policies to certain devices. The MUD manager only has the ability to elevate or restrict the network privileges of a device. Therefore, attacks on the MUD Manager cannot compromise devices, they can only enable a compromised device to access more of the network. Further security considerations related to the MUD Manager are covered in {{RFC8520}}.

If the MUD file and the software/firmware loaded onto the device gets out-of-sync a device may be firewalled and, with firewalling by networks in place, the device may stop functioning. This is, however,
not a concern specific to this specification but rather to the use of MUD in general. Below are two mitigations:

- A manufacturer must update the MUD file in advance of network service or product changes so 
that the new services can be supported. Because the MUD file is accessed by a URL means that it can be subsequently updated. This requires a MUD file being retrieved again. This handles the case when the device 
is already deployed and in use.

- There is a possibility that an IoT device has remained on-shelf inventory for an extended period, resulting in its MUD file being inaccessible at its previous location. This necessitates a decision on how to implement a fail-safe tailored to the particular environment.

# IANA Considerations

IANA is requested to add a new value to the SUIT manifest elements registry created with {{I-D.ietf-suit-manifest}}:

- Label: TBD1 [[Value allocated from the standards action address range]]
- Name: Manufacturer Usage Description (MUD)
- Reference: [[TBD: This document]]

--- back

# Acknowledgements
{: numbered="no"}

We would like to thank Roman Danyliw for his excellent review as the responsible security area director, Bahcet Sarikaya for his Genart review, Michael Richardson for his IoT directorate review and Susan Hares for her Opsdir review. During the IESG review Robert Wilton, Eliot Lear, Zaheduzzaman Sarker, Francesca Palombini, John Scudder, Paul Wouters, Éric Vyncke, and Murray Kucherawy.
