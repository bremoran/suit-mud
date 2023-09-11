---
title: Strong Assertions of IoT Network Access Requirements
abbrev: SUIT MUD Linkage
docname: draft-ietf-suit-mud-04
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

--- abstract

The Manufacturer Usage Description (MUD) specification describes the access and network functionality required for a device to properly function. The MUD description has to reflect the software running on the device and its configuration. Because of this, the most appropriate entity for describing device network access requirements is the same as the entity developing the software and its configuration.

A network presented with a MUD file by a device allows detection of misbehavior by the device software and configuration of access control.

This document defines a way to link a SUIT manifest to a MUD file offering a stronger binding between the two.

--- middle

# Introduction

Under {{RFC8520}}, devices report a MUD URL to a MUD Manager in the network, which then interacts with a MUD File Server
to ultimately obtain the MUD file. The following figure shows the MUD architecture.

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

RFC 8520 envisions different approaches for conveying the MUD URL from the device to the network such as:

- DHCP,
- IEEE802.1AB Link Layer Discovery Protocol (LLDP), and
- IEEE 802.1X whereby the URL to the MUD file would be contained in the certificate used in an EAP method.

The MUD Manager uses the MUD URL to fetch the MUD file, which contains connectivity-related functionality required for a device to properly function.

The MUD Manager must trust the MUD File Server from which the MUD file is fetched to return an authentic copy of the MUD file. This concern may be mitigated using the optional signature reference in the MUD file. The MUD Manager must also trust the device to report a correct MUD URL. In case of DHCP and LLDP the URL is likely unprotected. 

When the MUD URL is included in a certificate then it is authenticated and integrity protected. A certificate created for use with network access authentication is typically not signed by the entity that wrote the software and configured the device, which leads to a conflation of rights.

There is a need to bind the entity that creates the software/configuration to the MUD file because only that entity can attest the connectivity requirements of the device.

This specification defines an extension to the Software Updates for Internet of Things (SUIT) manifest format {{I-D.ietf-suit-manifest}} to include a MUD URL. When combining a MUD URL with a manifest used for software/firmware updates then a network operator can get more confidence in the description of the connectivity requirements for a device to properly function.

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all
capitals, as shown here.

# Workflow

The intended workflow is as follows:

* At the time of onboarding, devices report their manifest in use to the MUD Manager via attestation evidence in the Entity Attestation Token (EAT) {{I-D.ietf-rats-eat}}. 
    * Among other claims, the device will report its software digest(s), and the manifest URI in the EAT "manifests" claim to the MUD Manager. This approach assumes that attestation evidence includes a link to the SUIT manifest via the "manifests" claim (see Section 4.2.15 of {{I-D.ietf-rats-eat}}) and that this evidence can be carried in either a network access authentication protocol (for eample an EAP method) or some onboarding protocol like FIDO Device Onboard (FDO).
    * The MUD Manager can then (with the help of the Verifier) validate the evidence in order to check that the device is operating with the expected version of software and configuration.
    * Since a URL to the manifest is contained in the Evidence, the MUD Manager can look up the corresponding manifest.
* If the SUIT_MUD_container, see {{suit-extension}}, has been severed, the MUD Manager can use the suit-reference-uri to retrieve the complete SUIT manifest.
* The manifest authenticity is verified by the MUD Manager, which enforces that the MUD file presented is also authentic and as intended by the device software vendor.
* The MUD Manager acquires the MUD file from the MUD URL found in the SUIT manifest.
* The MUD Manager verifies the MUD file signature using the Subject Key Identifier (SKI) provided in the SUIT manifest.
* Then, the MUD Manager can apply any appropriate policy as described by the MUD file.

Each time a device is updated, rebooted, or otherwise substantially changed, it will execute the remote attestation procedures again.

# Operational Considerations

## Pros

The approach described in this document has several advantages over other MUD URL reporting mechanisms:

* The MUD URL is tightly coupled to device software/firmware version.
* The device does not report the MUD URL, so the device cannot tamper with the MUD URL.
* The onus is placed on the software/firmware author to provide a MUD file that describes the behavior of the software running on a device. 
* The author explicitly authorizes a key to sign MUD files, providing a tight coupling between the party that knows device behavior best (the author of the software/firmware) and the party that declares device behavior (MUD file signer).
* Network operators do not need to know, a priori, which MUD URL to use for each device; this can be harvested from the device's manifest and only replaced if necessary.
* A network operator can still replace a MUD URL in a SUIT manifest:

  * By providing a SUIT manifest that overrides the MUD URL.
  * By replacing the MUD URL in their network infrastructure.

* Devices can be quarantined if they do not attest a known software/firmware version.
* Devices cannot lie about which MUD URL to use.

## Cons

This mechanism relies on the use of SUIT manifests to encode the MUD URL. Conceptually, the MUD file is similar to a Software Bill of Material (SBOM) but focuses on the external visible communication behavior, which is essential for network operators, rather than describing the software libraries contained within the device itself. The SUIT manifest must then be conveyed to the network during onboarding or during the network access authentication step. To accomplish the transport of the manifest attestation evidence is used, which needs to be available at the protocol of choice. 

# Extensions to SUIT {#suit-extension}

To enable strong assertions about the network access requirements that a device should have for a particular software/configuration pair a MUD URL is added to the SUIT manifest along with a subject key identifier (ski).
The subject key identifier MUST be generated according to the process defined in {{I-D.ietf-cose-key-thumbprint}} and the SUIT_Digest structure MUST be populated with the selected hash algorithm and obtained fingerprint.
The subject key identifier corresponds to the key used in the MUD signature file described in Section 13.2 of {{RFC8520}}.

The following CDDL describes the extension to the SUIT_Manifest structure:

~~~CDDL
$$severable-manifest-members-choice-extensions //= (
  suit-manifest-mud => SUIT_Digest / SUIT_MUD_container
)
~~~

The SUIT_Envelope is also amended:

~~~CDDL
$$SUIT_severable-members-extensions //= (
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

This specification links MUD files to SUIT manifests for improving security protection and ease of use. By including MUD URLs in SUIT manifests an extra layer of protection has been created and synchronization risks can be minimized. If the MUD file and the software/firmware loaded onto the device gets out-of-sync a device may be firewalled and, with firewalling by networks in place, the device may stop functioning.

# IANA Considerations

IANA is requested to add a new value to the SUIT manifest elements registry created with {{I-D.ietf-suit-manifest}}:

- Label: TBD1 [[Value allocated from the standards action address range]]
- Name: Manufacturer Usage Description (MUD)
- Reference: [[TBD: This document]]

IANA is requested to add a new value to the SUIT envelope elements registry created with {{I-D.ietf-suit-manifest}}:

- Label: TBD2 [[Value allocated from the standards action address range]]
- Name: Manufacturer Usage Description (MUD)
- Reference: [[TBD: This document]]


