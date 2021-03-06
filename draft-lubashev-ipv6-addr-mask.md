---
title: A Recommendation for IPv6 Address/Mask Notation
abbrev: v6AddrMask
docname: draft-lubashev-ipv6-addr-mask-latest
date: {DATE}
category: info

ipr: trust200902
area: General
workgroup: IPv6 Operations
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
  compact: yes
  inline: yes
  text-list-symbols: -o*+


author:
 -
    ins: I. Lubashev
    name: Igor Lubashev
    org: Akamai Technologies
    email: igorlord@alum.mit.edu
 -
    ins: E. Nygren
    name: Erik Nygren
    org: Akamai Technologies
    email: erik+ietf@nygren.org
    uri: http://erik.nygren.org/


normative:

informative:
  OpenFlow:
    target: https://3vf60mmveq1g8vzn48q2o71a-wpengine.netdna-ssl.com/wp-content/uploads/2014/10/openflow-spec-v1.2.pdf
    title: OpenFlow Switch Specification, Version 1.2
    author:
      org: Open Networking Foundation
    date: December 5, 2011
    format:
      PDF: https://3vf60mmveq1g8vzn48q2o71a-wpengine.netdna-ssl.com/wp-content/uploads/2014/10/openflow-spec-v1.2.pdf
  TeraStream:
    target: https://ripe67.ripe.net/presentations/251-ripe2-4.pdf
    title: TeraStream - IPv6 Addressing Format
    author:
     -
      name: Peter Lothberg
      org: Deutsche Telekom
     -
      name: Mikael Abrahamsson
      org: Deutsche Telekom
    date: October 14, 2013
    format:
      PDF: https://ripe67.ripe.net/presentations/251-ripe2-4.pdf
  SURFnetAddrPlan:
    target: https://www.surf.nl/binaries/content/assets/surf/nl/kennisbank/2013/rapport_201309_ipv6_numplan_en.pdf
    target: https://www.ipv6forum.com/dl/presentations/IPv6-addressing-plan-howto.pdf
    title: Preparing an IPv6 Address Plan
    author:
      org: SURFnet
    date: September 18, 2013
    format:
      PDF: https://www.surf.nl/binaries/content/assets/surf/nl/kennisbank/2013/rapport_201309_ipv6_numplan_en.pdf
      PDF: https://www.ipv6forum.com/dl/presentations/IPv6-addressing-plan-howto.pdf
  IncognitoRoutingPlan:
    target: https://www.incognito.com/tips-and-tutorials/how-to-plan-routing-for-ipv6
    title: How to Plan Routing for IPv6
    author:
     -
      name: Andre Kostur
      org: Incognito
    date: July 2, 2015
  GeoAddressPatent:
    target: http://patft1.uspto.gov/netacgi/nph-Parser?patentnumber=7929535
    title: "US 7929535: Geolocation-based addressing method for IPv6 addresses"
    author:
     -
      name: Liren Chen
      org: Qualcomm Incorporated
     -
      name: Jack Steenstra
      org: Qualcomm Incorporated
     -
      name: Kirk S. Taylor
      org: Qualcomm Incorporated
    date: April 19, 2011

--- abstract

Since network operators are commonly assigned at least /48 IPv6
address prefixes, the operators frequently find opportunities to
devise addressing schemes that assign operational semantics to less
significant bit ranges.

This document extends the traditional address/prefix-length textual
representation to allow an interoperable represenation of groups of
IPv6 addresses sharing bit patterns that are not prefixes.  Devices
supporting the extended textual representation will allow operators to
implement policies based on such non-prefix address groups.




To implement policies based on bits in IPv6
addresses, the operators need a way to configure their equipment.
This RFC describes a textual representation of addresses sharing bit
patterns that are not prefixes.  Once an interoperable notation is
standartized and implemented by vendors, operators would find it
possible to configure policies they desire for their networks.

This RFC introduces IPv6 Address/Mask notation that is similar to the
IPv4 address/mask notation in its expressiveness, but it is derived
from the familiar address/prefix-length notation for clarity and
compatibility with existing parsers.

For example, using this representation, both 2001:db8::/32 and
2001:db8:://ffff:ffff:: have the same meaning.  However, a group of
addresses having the first 32 bits "2001:0db8::" and the last 16 bits
"::1234" requires the new representation:
2001:db8::1234//ffff:ffff::ffff or, equivalently,
2001:db8::1234//32+::ffff.


--- middle

Introduction     {#introduction}
============

We have learned to think of IPv4 address groupings in terms of CIDR
blocks, because virtually all logical address groupings fit that model
well: IP address allocations, subnets, routing announcements, etc.
With the move to IPv6, the primary mechanism for address grouping
remains matching by prefix length, albeit with longer prefix lengths.

While longer interface identifiers are useful for end user privacy
purposes, infrustructure networks and networks hosting content are
finding opportunities for assigning operator-specific semantics to bit
strings within addresses beyond the prefix.  Structured addressing
scheme for virtual services reduces complexity of managing addresses
and allows for concise policy definitions.

Numerous systems (see {{example-systems}} for examples) have been
assigning semantics to IPv6 bits that come after IANA prefix
bits. Developers of these systems attempted to communicate address
patterns underlying their system semantics both in documentation and
in machine-readable configurations accompanying the systems. Due to
the lack of a standard textual representation, the documentation often
resorted to pictographs and verbose English descriptions. The
configuration syntax and parsers were invariably ad hoc and
incompatible with other systems.

Here we define a syntax for representing groupings (matching rules) of
IPv6 addresses, where a set of less significant bits have a particular
value.  For example, 2001:db8::1234//ffff:ffff::ffff matches all
addresses whose 32 most significant bits are 2001:0db8 and whose 16
least significant bits are 0x1234.

The goal of this document is removing a hindrance to interoperability
of systems that wish to express rules and policies that apply to
address groups that cannot be expressed as CIDR blocks (see
{{example-systems}} for examples).  Guidance for the applicability of
such address groupings is outside the scope of this document.


Netmask and Prefix-Length Notations         {#current-notations}
-----------------------------------

There are two common textual representations for identifying groups of
addresses (networks, subnets, internet routing blocks). These
representations can also be used to identify an individual address and
its subnet.

The netmask notation described by {{!RFC0950}} is commonly used for
IPv4. It consists of a tuple of a network address and a network
mask. For example: 198.51.100.4 netmask 255.255.255.0.

The address/prefix-length notation described by {{!RFC4632}} is
commonly used for both IPv4 and IPv6. It consists of a tuple of a
network address and a prefix length. For example: 198.51.100.4/24 or
2001:db8::1234/32.

Depending on the context, netmask and prefix length notations can
specify either a "group of addresses" or "an individual address and a
group of addresses to which it belongs".  If the network address
contains one or more set bits not selected by the network mask or
prefix length, then network address specifies an individual address in
addition to the subnet.  For example: 198.51.100.4/24 means "address
198.51.100.4 within a group of addresses 198.51.100.0 -
198.51.100.255".


Problem Description    {#problem}
===================

The problem with the prefix length notation for IPv6 is that it is not
sufficiently expressive of IPv6 address groupings for a growing number
of applications.

IPv6 address allocation guidelines {{!RFC6177}} guarantee at least a
/48 allocation to network operators and strongly recommend a multi-/64
allocation to end sites. Because these address blocks are orders of
magnitude larger than any imaginable number of physical hosts, network
operators are managing those addresses in new and creative ways.

Sometimes, useful address grouping are not "all addresses that share a
prefix of a certain length".  Additionally, within an administrative
scope, there are use-cases where semantics are assigned to individual
bit ranges.

Consider these examples:

1. Allocating a block of addresses to each host and using the least
   significant bits to indicate a TLS certificate.  These operators
   may need a way to express a rule that applies to all traffic that
   uses a particular TLS certificate.

2. Network operators managing multiple similar data centers may have
   different prefixes routed to those data centers but desire a
   unified set of rules for assigning, managing, and routing IPv6
   addresses within those data centers. These operators need to
   express rules that do not depend on the prefixes of the addresses
   to which the rules apply.


Notational Conventions    {#conventions}
======================

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in {{!RFC2119}}.


IPv6 Address/Mask Textual Representation      {#addr-mask-representation}
========================================

This RFC extends address/prefix-length notation of {{!RFC4632}} in a
way that is reminiscent of the IPv4 netmask notation of
{{!RFC0950}}. The address/mask notation allows specifying IPv6 mask
instead of the prefix length.

The address/mask notation is defined as:

       ADDRESS // MASK

Both ADDRESS and MASK are IPv6 addresses.  The MASK indicates which
bits of the ADDRESS are relevant for the address grouping.  Note that
the MASK may be sparse and is not strictly a prefix. For example:
2001:db8::1234//ffff:ffff::ffff.

The "//" was chosen as a separator for address/mask notation, since it
is similar to the "/" separator used by address/prefix-length (and
hence is readily recognizable) but prevents incorrectly parsing
address/mask as address/prefix-length.

Constraints and Validation      {#validation}
--------------------------

To be a valid definition for just a group of addresses, the ADDRESS
part MUST NOT have any bits set outside of the MASK. Otherwise, the
ADDRESS // MASK represents an individual address and a group of
addresses it belongs to.

Examples: groups of addresses
-----------------------------

1. 2001:db8::1234//ffff:ffff::ffff

   This specifies IPv6 addresses that look like 2001:db8::1234 when
   you ignore bits 16-95.

2. ::aa00:1234//::ff00:ffff

   This specifies IPv6 addresses that have 0xaa in bits 24-31 and
   0x1234 in bits 0-15.

3. 2001:db8:://ffff:ffff::

   This is equivalent to 2001:db8::/32.

Examples: specific addresses and groups to which they belong
------------------------------------------------------------

1. 2001:db8::1:1234//ffff:ffff::ffff

   This specifies IPv6 address 2001:db8::1:1234 that belongs to a
   group of addresses that look like 2001:db8::1234 when you ignore
   bits 16-95.

2. 2001:db8::aa00:1234//::ff00:ffff

   This specifies IPv6 address 2001:db8::aa00:1234 that belongs to a
   group of addresses that have 0xaa in bits 24-31 and 0x1234 in bits
   0-15.

3. 2001:db8::1//ffff:ffff::

   This is equivalent to 2001:db8::1/32.

Textual representation of the Address and Mask
----------------------------------------------

When IPv6 mask is used after "//", both the network address and mask
parts MUST be formatted as IPv6 addresses and, therefore, their
canonical textual representation is dictated by {{!RFC5952}}.

Use prefix length instead of mask
---------------------------------

The canonical representation of a group of IPv6 addresses MUST use a
prefix length instead of a mask if possible.  That is, if the mask has
all its most significant bits set, up to some bit, followed by all
clear bits, then the canonical representation MUST use a prefix
length.


Scoped Mask/Value notation   {#scoped}
==========================

Since assigning operator-specific semantics to bit ranges is only
possible within the address space assigned to the operator by IANA, a
common use-case is to specify an address/mask within an IANA-assigned
prefix scope.  For example, all addresses ending with ::1234 within
2001:db8::/32 can be specified as 2001:db8::1234//ffff:ffff::ffff.

To make these representations easier to manage and validate, it helps
to have an explicit convention for representing prefixes within
address groups.  For example, 2001:db8::1234//ffff:ffff::ffff can be
represented as 2001:db8::1234//32+::ffff.

This is specified as:

        ADDRESS // PFX_LEN + SCOPED_MASK

Scoped Mask/Value notation representation can be canonicalized using a
ADDRESS // MASK notation. The canonical MASK is constructed by
performing the bitwise-or of SCOPED_MASK and the mask derived from an
address with the PFX_LEN most significant bits set.

The PFX_LEN most significant bits MUST NOT be set in SCOPED_MASK.


Compatibility and Parser guidelines   {#compatibility}
===================================

Only parsers that wish to support address groupings that cannot be
represented using address/prefix-length are required to support
address/mask notation.

Systems that support communicating address grouping in address/mask
notation to other systems SHOULD communicate such grouping in
canonical address/prefix-length notation, if possible.  This ensures
compatibility with systems that do not support address/mask notation,
if all configured address groupings are proper CIDR prefixes.

Address groupings that cannot be expressed using address/prefix-length
notation MAY be communicated using Scoped Mask/Value ({{scoped}})
notation, as long as the PFX_LEN (semantic prefix scope) has been
configured via external means (i.e. PFX_LEN SHOULD NOT be
automatically derived from the MASK bitmap by the system itself).


Security Considerations    {#security}
=======================

This document only defines textual representation for IPv6 address
groupings.  It does not intend to recommend when assigning semantics
to specific bit ranges and matching based on bit substrings is
applicable or appropriate.

IP addresses can be spoofed or attacker-controlled.  This is
especially true of IPv6 addresses differing only in less significant
bits and belonging to different administrative domains.  When used in
policies applied to incoming traffic, the MASK part of the
address/mask notation SHOULD have as many set bits as the semantics of
the policy would allow.

Operators wishing to assign semantics to bit ranges should be aware
that these semantics may be guessable or leaked outside the
organization. Hence, there is a risk of privacy/information leakage.


Address Utilization Considerations  {#utilization}
==================================

Since IPv6 allocation guidelines {{!RFC6177}} guarantee at least a /48
allocation to network operators, it would be an enormous waste of the
address space to assign IPv6 addresses only to physical hosts or
network interfaces.

On the other hand, a gratuitous use of lower address bits can lead to
a premature address space exhaustion and difficulties in adapting to
the future needs of the organization within the assigned address
space.  An example of such gratuitous use is designating large parts
of the address space for a bitmask, where only a small fraction of all
possible bit combinations is utilized.


IANA Considerations   {#iana}
===================

This document has no actions for IANA.


Change Log
==========

Since draft-lubashev-ipv6-addr-mask-00
--------------------------------------

- Changed separator from "/" to "//"
- Addressed privacy in {{security}}
- Added {{compatibility}}
- Added {{utilization}}
- Added {{example-systems}}


Acknowledgments
===============

The Acknowledgments will come here.


--- back


Examples of Semantic Use of Lower Address Bits   {#example-systems}
==============================================

Assigning semantics to lower bits of IPv6 addresses and defining
policies based on such address groupings have been done for many
years. Documents describing such policies and configurations for
equipment implementing these policies do not use a consistent
notation.  Most documents resort to pictographs and verbose English,
while the configuration syntax and parsers are invariably ad hoc.


A Framework for Semantic IPv6 Prefix and Gap Analysis
---------------------------------------------------------------------

Internet draft {{?I-D.jiang-semantic-prefix}} describes the need for
adding semantics to lower IPv6 address bits to define address groups
that cannot be expressed as CIDR blocks and analyzes some implications
of this practice. This draft describes uses of such address groups for
creating routing policies as well as configuring such policies on
hosts and routers both statically and dynamically.


Teredo
------

Teredo protocol {{?RFC4380}} uses four bit ranges past Teredo IANA
prefix bits to encode server and client IPv4 addresses, a flags
bitmap, and a port (section "4. Teredo Addresses").


OpenFlow Switch Configuration
-----------------------------

OpenFlow Switch Specification {{OpenFlow}} describes OpenFlow switch
configuration API that can match flows based on an arbitrary IPv6
bitmask applied to IPv6 source (OXM_OF_IPV6_SRC) or destination
(OXM_OF_IPV6_DST) addresses. Version 1.2 was the first version to
introduce such IPv6 address/bitmask flow match rules (chapter A.2.3.7)
in 2011.


TeraStream IPv6 Addressing
--------------------------

TeraStream {{TeraStream}} system is using bit ranges to encode service
type in IPv6 address bits that come after IANA prefix bits. The system
was launched in 2012.


SURFnet IPv6 Address Plan and Incognito Routing Plan
----------------------------------------------------

SURFnet published a white paper {{SURFnetAddrPlan}} advocating that
ISPs use bit ranges past their IANA prefix to encode geo-location and
address use types. The white paper is giving examples of a sample
address allocation that uses one nibble (bits 68-71) for encoding
geo-location and another nibble (bits 64-67) for the use type.

IPv6 Routing Plan {{IncognitoRoutingPlan}} by incognito is advocating
allocating bit ranges past an IANA prefix to designate various address
attributes, including "subnet types".


Geolocation-based addressing method for IPv6 addresses
------------------------------------------------------

Qualcomm US patent 7,929,535 {{GeoAddressPatent}} describes a method
of embedding geo-location information, such as latitude, longitude,
altitude, in predefined ranges of bits of an IPv6 address past their
IANA prefix.


Customer IDs in less significant bits
-------------------------------------

CDNs and hosting providers host web sites belonging to multiple
customers using shared servers.  Due to the lack of support for SNI
TLS extension {{?RFC6066}} by some user agents active on the Internet,
CDNs resort to using unique IP addresses to identify specific customer
domains and, hence, certificates for TLS negotiation.  In case of IPv6
addresses, at least some CDNs use the less significant bits of an IPv6
address to identify customer domains (while the more significant bits
carry internal routing information).  The configuration of systems
matching lower bits of IPv6 addresses to individual customer domains
must use ad hoc syntax due to the lack of a standard way to express
semantics of matching on bit ranges other than address prefixes.

