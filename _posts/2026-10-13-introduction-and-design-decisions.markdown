---
layout: post
title:  "Introduction and design decisions"
date:   2026-10-13 20:27:00 -0500
categories: design-decisions
---
My name is Don, the (as of writing, only) maintainer of the Gwydion project. I do not have formal education on any kind of computer science course and my current professional career is in the frontline operations of a United States airline. However, many years before my current career I've had experience in web design and development and owned a very small web design and eBook binding company in the mid-to-late 2000's. It's understandable you'd find it strange that someone like me is attempting to create a platform for identity document management and validation, and I'd like to use this blog to explain my reason and the current decisions I've made about the project.

## Why Gwydion?

First and foremost, my interest in Gwydion came from an overlap of my current profession and the furry community. Through my job and my hobbies I've made many friends inside and outside my line of work. My group of friends, while I can never see them in person as much as I like, are passionate, quirky, funny, and bring me so much joy even if I'm only watching through a cabin window or living vicariously through their messages and social media posts. Our connections was a salve to an extreme loneliness that is unique to the aviation industry.

I've never thought of myself good at coding and thought my strength was in design. Using my strengths, I read tutorials and struggled through code to create a generator for furry convention badges that mimicked the look of travel documents defined in [ICAO Doc 9303][icao9303]. Every travel document I could find in ICAO's documentation my generators could create. I am a perfectionist though, and--for a lack of better word--the generators created just for fun were quite overenginnered.

Through reading ICAO's documentation, my generators did more than mimic travel documents in appearance:

* Machine-Readable Zone (MRZ) check digits were dynamically calculated based on what someone entered
* Visual Digital Seals (VDS) structured the data in the exact binary payload a scanner expects
* Rendered badges could be read by smartphone document readers like [Regula Document Reader][regula]

In short, while no automated border control gate or airport passport reader **should** accept anything made by the generators because of using a user-defined authority code range and the lack of an actual signature payload (the "signatures" generated were just random bytes), the actual badges created by the generators followed the specifications accurately enough that they could be read by smartphone document readers and could be read by other readers following the specification.

### Convention "reg-con"

While I was happily making travel document themed badges for my friends, I was also attending conventions, and I was noticing a problem: convention lines, whether anime, furry, or otherwise were way too long. What is preventing the kinds of systems we have in place in passenger terminals, where people can easily enter with the scan of a digital boarding pass or some other credential and basically walk-in without waiting in a ticketing line, from being implemented in furry conventions? Especially since specifications exist, can registration lines be shortened through an identity platform?

Those questions ran through my head as I looked over the code of my furry badge generators once more. Fun aesthetic badges aside, I had generators for identity documents that adhered to a standard. What was missing was:

* Organizations that would accept a particular identity document
* A mechanism to sign the document's digital binary payloads

Gwydion is my attempt to provide those missing pieces.

## So, what exactly will be Gwydion?

Stripped to essentials, Gwydion's core will be a platform that provides a cryptographic signature to a physical document. That's it. Its main functions will be:

* Approve and cryptographically sign a document
* Provide a validation client the public keyring it uses to sign documents
* Provide a list of revoked keys in the public keyring
* Provide a list of revoked (lost, stolen, damaged, etc.) documents

The platform needs to be more than just those four points for it to be a platform anyone would want to use. Organizations who want to use Gwydion would also want a way to style and brand their documents, print their documents, manage keys, and many other things. However it doesn't change the core.

## What benefits does the platform provide?

Quoting [Part 13][part13] of ICAO Doc 9303, the technical benefits of using Gwydion are:

> * _Asymmetry_. Due to using asymmetric cryptography, the cost of attaching a digital seal is considerably higher than the cost of issuing a document protected with a digital seal.
>
> * _Personalization_. Each digital seal verifies the information printed on the physical document ... There is no direct equivalent of a blank document, therefore no blanks can be lost or stolen.
>
> * _Easy verification_. Even untrained persons are able to verify a document protected with a digital seal by using low-cost equipment, such as an application on a smartphone.

Translated, Gwydion would make it either expensive or difficult to forge an identity document, and would make it easy to verify its validity through low-cost _offline_ equipment, in a non-proprietary open method standardized by an international body and with free/open-source software.

These benefits can be valuable for any organization, but let's return to furry conventions:

### Registration line congestion

With Gwydion, furry convention non-profits can immediately issue badges to convention participants, with an (optional) ID/Know-Your-Customer (<abbr>KYC</abbr>) verification to certify access to 18+ panels or areas, during online registration. After registration is completed, the details on the physical badge are signed by Gwydion. The generated visual digital seal can be printed on a physical badge to be mailed to the participant, or "printed" on a digital badge provided through a convention's app.

Lengths of physical lines are reduced to people who were unwilling to do online ID/KYC to access age-restricted areas, people who are registering day-of, and people desiring/needing a physical badge (like a fursuiter) that were unable to receive a mailed one.

### 100% offline validation checks

Convention staff and volunteers can continue to resume operations as they were, simply inspecting badges through visual alone for standard areas like entering an unrestricted panel or actual convention premises. As ID/KYC verification was done at registration, the only identification participants need to present to age-restricted areas is their con badge. Staff may also choose to do spot-checks on participants randomly to verify their badges are valid.

Importantly, as the verification uses asymmetric cryptography, Gwydion's public keys can simply be downloaded locally into the validating device (like a convention-provided smartphone) at the start of the day. This allows badge verification to stay in effect independent of the performance of (usually-very-spotty) convention center and hotel Wi-Fi.

### Identity document federation

As the platform works on cryptographic keys, this can allow identity documents or "visa" (event) documents to be issued by one organization and chosen to be accepted by another organization.

A furry convention organizer in Detroit, Michigan, could partner with a smaller organization in Detroit's metro area to issue an event document valid for a year. Each organization can issue that document based on their specifications and independent of the other organization. At events where those kinds of documents are valid, the only thing needed for a validator to check validity is the public keys that issued the documents from both organizations.

### No leak of personally-identifiable information (PII)

While Gwydion _can_ store personally-identifiable information (<abbr>PII</abbr>) for organizations that may not have their own identity management system, ideally Gwydion will have no further need of any information about a person after a document has been signed and printed. All the information needed for verification is in the printed document itself and no storage of that information is necessary.

In a situation where Gwydion is not being used as an identity store, the only information Gwydion has are document numbers and keys. A total compromise of Gwydion in that situation would be the loss of document numbers and potentially private key material stored locally, but no PII will leak as a result. This is important for the furry community and other subculture communities where someone's legal name may be different than the name they use in the community.

### A free and open source platform

I intend Gwydion to be free as in speech and free as in beer, and commit to continue working on it as long as I and/or others find value in the project. This is significant because while components of identity document platforms exist in the free/open-source community, a platform unifying it into a usable turnkey product does not.

Additionally, small and medium-sized organizations that require some form of identity document issuance are stuck with two essentially bad choices: manual printing of documents and logging them in spreadsheets/databases, or cost-prohibitive enterprise vendors that charge per-scan or per-issuance fees. Gwydion provides a choice based on an open international specification that anyone can improve on and that everybody can own and use as they see fit.

## What are Gwydion's limitations?

No platform is perfect, and Gwydion has real threats unsolvable by technology alone (i.e. inherent risks) that organizations must think about. The most significant risks are:

1. _Photo substitution and document counterfeiting_. While Gwydion's use of VDS on a physical or digital identity document prevents document forging (that is, prevents someone from creating a document from scratch) because of the cryptographic signature, it does not prevent against making a complete clone or someone replacing a photo (which is generally too big to be placed in the VDS) on a document with a valid VDS. In the future active authentication/chip access control is possible to implement which can mitigate this inherent risk. For now, partial mitigations include ensuring people know what to check for when looking at a document, especially any physical security features like holograms, microprinting, or ultraviolet ink present.

2. _Trusted insiders and rogue operators_. Authorized users with legitimate access to Gwydion's administrative interface or server/container can attempt to submit fraudulent records for signing or compromise Gwydion's keys. This threat isn't unique to Gwydion but must be considered when deploying the platform. Using the platform safely and with trust may require examining an organization's procedural policy to mitigate this threat.

Consequently, the kinds of organizations that would be comfortable with Gwydion are those where human physical inspection or low incentive fraud boundaries provide enough security. Furry convention organizers, anime convention organizers, small and medium enterprises, non-governmental or humanitarian organizations, and local governments using Gwydion for secondary credentials (like library cards) are examples of organizations that may be comfortable with these risks. High-security facilities, financial institutions, and national governments may be uncomfortable or outright reject Gwydion so long as its validation model uses MRZ/VDS only for protection.

## Where is Gwydion right now?

As of right now, I've tidied up [@gwydion-id/icao-9303-models][icao-models] in preparation to write the main server component for the Gwydion platform. I have design notes and diagrams in my personal notes about how I want the architecture to look and what APIs the server will provide to the clients that will use it. Clients, in my design notes, will be based on capabilities, meaning that there will be an administrative client (to manage the server), an issuing client (one that can send documents for approval and signing), a templating client (one that formats and styles documents and the data each document requires), a public key infrastructure (<abbr>PKI</abbr>) "server" client (one that shares public keys and key/document revocation lists), and a validator client (one that does the actual offline scanning of documents and determines its validity). This design additionally means the only clients needed "in the open" are the PKI "server" client and the validator client, and losing those clients through security compromise does not compromise the rest of the system.

None of the server or client code has been published to GitHub yet as I'm still fleshing out the design. My head works better with code if I know what 'something' looks like and setting the APIs help me create its shape and discourage me from veering away from it.

## Thanks for reading this far

I hope to continue updating here on Gwydion's progress, though I can't promise fixed timelines or milestones as I'm doing this project by myself. I look forward to getting things up sooner than later though and plan to keep logging my progress.

[icao9303]: https://www.icao.int/publications/doc-series/doc-9303
[regula]: https://play.google.com/store/apps/details?id=com.regula.documentreader&hl=en_US&pli=1
[part13]: https://www.icao.int/sites/default/files/publications/DocSeries/9303_p13_cons_en.pdf
[icao-models]: https://www.npmjs.com/package/@gwydion-id/icao-9303-models
