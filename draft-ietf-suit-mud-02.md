---
title: Strong Assertions of IoT Network Access Requirements
abbrev: SUIT MUD Linkage
docname: draft-ietf-suit-mud-02
category: std

ipr: pre5378Trust200902
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
      organization: Arm Limited
      email: hannes.tschofenig@gmx.net

normative:
  RFC7093:
  RFC8520:
  RFC2119:
  RFC8174:
  I-D.ietf-rats-eat:
  I-D.ietf-suit-manifest:

--- abstract

The Manufacturer Usage Description (MUD) specification describes the access and network functionality required a device to properly function. The MUD description has to reflect the software running on the device and its configuration. Because of this, the most appropriate entity for describing device network access requirements is the same as the entity developing the software and its configuration.

A network presented with a MUD file by a device allows detection of misbehavior by the device software and configuration of access control.

This document defines a way to link a SUIT manifest to a MUD file offering a stronger binding between the two.

--- middle

# Introduction

Under {{RFC8520}}, devices report a URL to a MUD manager in the network. RFC 8520 envisions different approaches for conveying the information from the device to the network such as:

- DHCP,
- IEEE802.1AB Link Layer Discovery Protocol (LLDP), and
- IEEE 802.1X whereby the URL to the MUD file would be contained in the certificate used in an EAP method.

The MUD manager then uses the the URL to fetch the MUD file, which contains access and network functionality required a device to properly function.

The MUD manager must trust the service from which the URL is fetched and to return an authentic copy of the MUD file. This concern may be mitigated using the optional signature reference in the MUD file. The MUD manager must also trust the device to report a correct URL. In case of DHCP and LLDP the URL is unprotected. When the URL to the MUD file is included in a certificate then it is authenticated and integrity protected. A certificate created for use with network access authentication is typically not signed by the entity that wrote the software and configured the device, which leads to conflation of local network access rights with rights to assert all network access requirements.

There is a need to bind the entity that creates the software and configuration to the MUD file because only that entity can attest the network access requirements of the device. This specification defines an extension to the SUIT manifest to include a MUD file (per reference or by value). When combining a manufacturer usage description with a manifest used for software/firmware updates (potentially augmented with attestation) then a network operator can get more confidence in the description of the access and network functionality required a device to properly function.

# Terminology

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
   "OPTIONAL" in this document are to be interpreted as described in
   BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all
   capitals, as shown here.

# Architecture

The intended workflow is as follows:

* At the time of onboarding, devices report their manifest in use to the MUD manager.
* If the SUIT_MUD_container has been severed, the suit-reference-uri can be used to retrieve the complete manifest.
* The manifest authenticity is verified by the MUD manager, which enforces that the MUD file presented is also authentic and as intended by the device software vendor.
* Each time a device is updated, rebooted, or otherwise substantially changed, it will execute an attestation.
    * Among other claims in the Entity Attestation Token (EAT) {{I-D.ietf-rats-eat}}, the device will report its software digest(s), configuration digest(s), primary manifest URI, and primary manifest digest to the MUD manager.
    * The MUD manager can then validate these attestation reports in order to check that the device is operating with the expected version of software and configuration.
    * Since the manifest digest is reported, the MUD manager can look up the corresponding manifest.
* If the MUD manager does not already have a full copy of the manifest, it can be acquired using the reference URI.
* Once a full copy of the manifest is provided, the MUD manager can verify the device attestation report and apply any appropriate policy as described by the MUD file.

# Extensions to SUIT

To enable strong assertions about the network access requirements that a device should have for a particular software/configuration pair, we include the ability to add MUD files to the SUIT manifest. However, there are also circumstances in which a device should allow the MUD to be changed without a firmware update. To enable this, we add a MUD url to SUIT along with a subject-key identifier, according to {{RFC7093}}, mechanism 4 (the keyIdentifier is composed of the hash of the DER encoding of the SubjectPublicKeyInfo value).

The following CDDL describes the extension to the SUIT_Manifest structure:

~~~CDDL
? suit-manifest-mud => SUIT_Digest
~~~

The SUIT_Envelope is also amended:

~~~CDDL
? suit-manifest-mud => bstr .cbor SUIT_MUD_container

SUIT_MUD_container = {
    ? suit-mud-url => #6.32(tstr),
    ? suit-mud-ski => SUIT_Digest,
    ? suit-mud-file => bstr
}
~~~

The MUD file is included verbatim within the bstr. No limits are placed on the MUD file: it may be any RFC8520-compliant file.

# Security Considerations

This specification links MUD files to other IETF technologies, particularly to SUIT manifests, for improving security protection and ease of use. By including MUD files (per reference or by value) in SUIT manifests an extra layer of protection has been created and synchronization risks can be minimized. If the MUD file and the software/firmware loaded onto the device gets out-of-sync a device may be firewalled and, with firewalling by networks in place, the device may stop functioning.

# IANA Considerations

suit-manifest-mud must be added as an extension point to the SUIT manifest registry.
