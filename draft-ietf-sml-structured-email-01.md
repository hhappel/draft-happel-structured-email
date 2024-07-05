%%%
title = "Structured Email"
area = "ART"
workgroup = "SML"

[seriesInfo]
name = "Internet-Draft"
value = "draft-ietf-sml-structured-email-01"
stream = "IETF"
status = "standard"

[[author]]
initials = "H.-J."
surname = "Happel"
fullname = "Hans-Joerg Happel"
organization = "audriga GmbH"
  [author.address]
   email = "happel@audriga.com"
   uri = "https://www.audriga.com"

%%%

.# Abstract 

This document specifies how a machine-readable version of the content of email messages can be added to those messages.

{mainmatter}

# Introduction

Information on websites and in email messages mostly addresses human readers. However, various attempts have been made to make such information - fully or in part - machine-readable, so that tools can assist users in dealing with that information more efficiently.

One widespread approach is the usage of [@SchemaOrg] vocabulary which can be embedded in the HTML markup of websites. It is then crawled by search engines to provide web search result snippets in a more advanced form (such as displaying ratings, opening hours, or contact information).

Similarly, some web shops and airlines include Schema.org vocabulary in their receipt email messages.  Some email providers and open source tools extract and use this data. However their implementations differ in many details.

The goal of this specification is to provide a clear and comprehensive specification for this practice and to provide ground for potential future extensions.

# Conventions Used in This Document

The terms "message" and "email message" refer to "electronic mail messages" or "emails" as specified in [@RFC5322]. The term "Message User Agent" (MUA) denotes an email client application as per [@RFC5598].

The terms "machine-readable data" and "structured data" are used in contrast to "human-readable" messages and denote information expressed "in a structured format (..) which can be consumed by another program using consistent processing logic" [@MachineReadable].

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [@RFC2119] [@RFC8174] when, and only when, they appear in all capitals, as shown here.

# Representing structured data 

In order to exchange structured data, one needs to chose a formal language and a serialization format.

In a decentralized setup, (shared) vocabularies can be used based on this this.

## Knowledge representation language

The Resource Description Framework ([@RDF]) is a  formal language for knowledge representation standardized by the W3C. It is already used for annotating websites and emails, as it is underlying [@SchemaOrg]. Among the various serializations for RDF, JSON-LD ([@JSONLD]) has become the most commonly used serialization used on websites [@WDCStats].

Hence, structured data in email messages SHOULD be expressed in the JSON-LD serialization of RDF.

	For discussion, see also:
	https://github.com/hhappel/draft-happel-structured-email/issues/1

## Vocabularies

Using RDF/JSON-LD, users are free to express any kind of information in structured data. For reuse and reference however, it is common to agree upon certain core concepts/entities and properties for a certain domain. Those are typically collected and shared in so-called vocabularies.

One such vocabulary, that has been intially designed for usage on websites, is the [@SchemaOrg] vocabulary. A small part of this vocabulary is used by email senders and processed by email providers already.

Users that want to add structured data into email message SHOULD use concepts from [@SchemaOrg], if they fit their use case. They MAY however use any valid JSON-LD. 

Similar to the case of arbitrary MIME body parts, which might not be supported by every MUA, interoperability can still be achieved, either by extending a MUA, or by handling data outside of the MUA.

	For discussion, see also:
	https://github.com/hhappel/draft-happel-structured-email/issues/2

# Structured data in email messages

This section discusses the placement of structured data within email messages and identifiers for referencing between structured data and other parts of a message.

## Placement

This document targets structured data describing the content of an email message itself. Since users may add arbitrary structured data (e.g., as MIME body parts of type "application/ld+json") to an email message, we need to define which kinds of structured data are supposed to be representative of the email message content.

For this, we distinguish the cases of full, partial, and-non representation.

	For discussion, see also:
	https://github.com/hhappel/draft-happel-structured-email/issues/3

### Full representation

If structured data is intended by the sender to fully describe the human readable content of an email message, it MUST be added as a multipart/alternative entity with the content type "application/ld+json".

The email message SHOULD in this case also contain a "text/plain" and a "text/html" version of the content.

### Partial representation

If structured data is intended to describe only a subset of the human-readable content, it must be enclosed in a "script" HTML tag within the HTML "body" tag of the text/html body part of the email message (see example at the end).

### Non-representation

In the case of non-representation, there is no relation between structured data and the human readable content.

While this is certainly a special case, an example would, e.g., be "preemptive" structured vacation notices as described in [I-D.happel-structured-vacation-notices-00].


## Identifiers

There are existing use cases for cross-referencing between different parts of a MIME message, for which 
[@RFC2392] defines the "cid:" and "mid:" URI schemes.

In a similar fashion, cross-referencing might occur between structured data and other message parts.

### Using identifiers in structured data

Most nodes and properties in JSON-LD are identified using IRIs [@RFC3987]. Since any [@RFC2392] (cid/mid) reference forms a valid IRI, those references can be directly used in JSON-LD.

There are two main cases for which "cid:"-identifiers SHOULD be used in structured data.

First, if structured data references binary content such as images or other files which already exist as MIME body parts within the same message.

If a "cid:" value is used in a JSON-LD "@id" property, the corresponding JSON-LD node can be considered to describe the MIME body part identified by that "cid:". This MAY be used to denote that certain structured data is explictily describing that MIME body part. This MUST NOT be used for the main text/plain or text/html body parts, though.

	For discussion, see also:
	https://github.com/hhappel/draft-happel-structured-email/issues/4

### Using structured data identifiers in text/html

In the case of "partial representation", a MUA will still primarily display the human readable part of a message.

It might however be helpful if the MUA is able to understand which parts of human readable text refer to certain structured data.

For this purpose, the sender may add a HTML "data-id" property ([@HTMLData]) to any HTML entity in the text/html body, which references the "@id" property of a JSON-LD node in the structured data.

Besides referencing the corresponding JSON-LD node, a sender might also want to denote if the underlying data is "extensively" described or just mentinoed in the human readable representation. Depending on that, a MUA might provide different additional visualizations for the user.´

	For discussion, see also:
	https://github.com/hhappel/draft-happel-structured-email/issues/5

# Structured data across email messages

## Forwarding

Forwarding messages including structured data needs to be considered from a privacy perspective, particularly in cases of "non-representation", when the user has no way to determine structured data from the human readable part of the message.

A MUA MUST strip non-representative structured data when forwarding messages. Note that this does only apply to MUAs directed by users and not for automated forwarding set up by a user.

	For discussion, see also:
	https://github.com/hhappel/draft-happel-structured-email/issues/6

## Replies

In order to allow responses to structured email messages, the [@SchemaOrg] vocabulary specifies a property called "potentialAction" ([@PotentialAction]).

If the "target" property of an action points to a "mailto:" URI, the email user agent SHOULD reply with a structured email if the user triggers the action.

	For discussion, see also:
	https://github.com/hhappel/draft-happel-structured-email/issues/7

## Error replies

An original sender may not assume that a structured email has been processed by a recipient. However, if a recipient replies to a structured email, the original sender MAY want to signal an issue with the response, such as if a contradicting response has already been received.

In this case, a "full representation"-style error message MAY be returnend to the sender of the erroneous response. Example: TBD

	For discussion, see also:
	https://github.com/hhappel/draft-happel-structured-email/issues/8

## Updates

In human-readable messages, human language can be used to update or recall information that was conveyed in prior messages. Accordingly, there needs to be a machine-readable mechanism that allows to express the update or recall of information of structured data.

Structured data SHOULD be updated, if a later email message with a SUPERSEDES header field ([@RFC4021]; "superseding messing") referencing the message id of the original email message is processed. In this case, structured data of the original message should be fully revoked and replaced by the structured data of the superseding message (which might be empty).

Structured data in a superseding message MUST be ignored if:

* Structured data from the original message is not or cannot be revoked
* In particular, if the original message has already been replied to by the recipient

	For discussion, see also:
	https://github.com/hhappel/draft-happel-structured-email/issues/9

# Header fields and message flags

This sections presents header fields and IMAP flags which are supposed to support MUAs in dealing with structured email.

## Presence of structured data

In some use cases, MUAs might benefit from information about message details without having to evaluate the full message body. 

For example, the "$hasAttachment" IMAP flag ([@HasAttachment]) was proposed to signal the existence of MIME attachments in a message.

The following procedures should apply to structured email.

A sending MUA (aMUA) SHOULD add a header field  "Structured data" if a message contains structured data. The value for this field MUST include one of the following values (case-insensitive):

* "Full" for full representation
* "Partial" for partial representation

The "Structured data" fields SHOULD additionally include (case-insensitive, comma-separated) the value "Action", if a message contains a "potentialAction" a MUA might want to investigate.

Similarly, the IMAP flags "$hasStructuredData" and "$hasStructuredDataAction" MAY be used, if an inbound message is found to contain structured data, but neither of the aforementioned header fields.

	For discussion, see also:
	https://github.com/hhappel/draft-happel-structured-email/issues/10

## Action processing

A structured email can contain "potentialActions". MUAs need to ensure that such actions are not triggered multiple times - either within the same MUA or across multiple concurrent MUAs.

For this purpose, the "\Answered" flag ([@RFC9051]) is not appropriate, as it has an established meaning and implementations for regular, manually authored responses.

Therefore, a MUA MUST set a flag "$structuredDataActionSent" if a potentialAction has been responsed to - either by the user or some other mechanism on behalf of the user.

	For discussion, see also:
	https://github.com/hhappel/draft-happel-structured-email/issues/11

# Examples

## Partial representation

Pleacement of JSON-LD markup in a "text/html" body part:

	<html>
	<body>
	<script language="ld+json">
	...
	</script>
	</body>
	</html>


# Security and trust

Email user agents that want to support structured email should follow guidance to ensure trust and security standards. These will be elaborated in a separate specification.

# Implementation status

< RFC Editor: before publication please remove this section and the reference to [@RFC7942] >

This section records the status of known implementations of the protocol defined by this specification at the time of posting of this Internet-Draft, and is based on a proposal described in [@RFC7942]. The description of implementations in this section is intended to assist the IETF in its decision processes in progressing drafts to RFCs. Please note that the listing of any individual implementation here does not imply endorsement by the IETF. Furthermore, no effort has been spent to verify the information presented here that was supplied by IETF contributors. This is not intended as, and must not be construed to be, a catalog of available implementations or their features. Readers are advised to note that other implementations may exist.

According to [@RFC7942], "this will allow reviewers and working groups to assign due consideration to documents that have the benefit of running code, which may serve as evidence of valuable experimentation and feedback that have made the implemented protocols more mature. It is up to the individual working groups to use this information as they see fit".

## Structured Email plugin for Roundcube Webmail

An open source plugin for the Roundcube Webmail software is developed to serve as a reference implementation for this specification ([@RC-SML]).

Beyond that, some ISPs and open source tools provide implementation partly compliant with this specficiation ([@SchemaOrgEmail]).

# Security considerations

See section "security and trust".

# Privacy considerations

See section "security and trust".

# IANA Considerations

This document has no IANA actions at this time.

(TBD IMAP flags)


{backmatter}

<reference anchor="RDF" target="https://www.w3.org/TR/rdf11-concepts/">
   <front>
      <title>RDF 1.1 Concepts and Abstract Syntax</title>
      <author>
        <organization>W3C RDF Working Group)</organization>
      </author>
      <date>2014</date>
   </front>
</reference>

<reference anchor="JSONLD" target="https://www.w3.org/TR/json-ld/">
   <front>
      <title>JSON-LD 1.1</title>
      <author>
        <organization>W3C JSON-LD Working Group</organization>
      </author>
      <date>2020</date>
   </front>
</reference>


<reference anchor="WDCStats" target="http://webdatacommons.org/structureddata/#toc3
">
   <front>
      <title>Web Data Commons - Microdata, RDFa, JSON-LD, and Microformat Data Sets</title>
      <author>
        <organization>Web Data Commons Project</organization>
      </author>
      <date>2022</date>
   </front>
</reference>

<reference anchor="MachineReadable" target="https://csrc.nist.gov/glossary/term/Machine_Readable">
   <front>
      <title>NIST IR 7511 Rev. 4</title>
      <author>
        <organization>NIST</organization>
      </author>
      <date>2016</date>
   </front>
</reference>

<reference anchor="SchemaOrg" target="https://schema.org/">
   <front>
      <title>Schema.org</title>
      <author>
        <organization>W3C Schema.org Community Group</organization>
      </author>
      <date>2023</date>
   </front>
</reference>

<reference anchor="RC-SML" target="https://github.com/audriga/roundcube-structured-email/">
   <front>
      <title>Structured Email plugin for Roundcube Webmail</title>
      <author>
        <organization>audriga GmbH</organization>
      </author>
      <date>2024</date>
   </front>
</reference>

<reference anchor="SchemaOrgEmail" target="https://structured.email/content/related_work/frameworks/schema_org_for_email.html">
   <front>
      <title>Schema.org for email</title>
      <author>
        <organization>Structured Email</organization>
      </author>
      <date>2023</date>
   </front>
</reference>

<reference anchor="PotentialAction" target="https://schema.org/potentialAction">
   <front>
      <title>Schema.org: potentialAction</title>
      <author>
        <organization>W3C Schema.org Community Group</organization>
      </author>
      <date>2023</date>
   </front>
</reference>

<reference anchor="HTMLData" target="https://html.spec.whatwg.org/multipage/dom.html#attr-data-*">
   <front>
      <title>HTML Living Standard: Embedding custom non-visible data with the data-* attributes</title>
      <author>
        <organization>WHATWG</organization>
      </author>
      <date>2023</date>
   </front>
</reference>

<reference anchor="HasAttachment" target="https://mailarchive.ietf.org/arch/msg/imapext/MVE5eNHOaNIVGUvN1RKtBL8b278/">
   <front>
      <title>Registering $hasAttachment &amp; $hasNoAttachment</title>
      <author>
        <organization>IETF imapext WG mailing list</organization>
      </author>
      <date>2018</date>
   </front>
</reference>


