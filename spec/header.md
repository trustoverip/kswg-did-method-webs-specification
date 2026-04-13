ToIP `did:webs` Method Specification
==================

**Specification Status**: v0.10.3

In order to further validate and improve the specification and to
demonstrate interoperability between multiple implementations of did:webs,
we encourage additional did:webs implementations to the original
[did:webs Reference Implementation](#DIDWEBS-REF-IMPL).

**Latest Draft:**

[https://github.com/trustoverip/kswg-did-method-webs-specification][kswg-gh]

[kswg-gh]: https://github.com/trustoverip/kswg-did-method-webs-specification

**Editors:**

- [Phil Feairheller](https://github.com/pfeairheller), [GLEIF](https://gleif.org)
- [Daniel Hardman](https://www.linkedin.com/in/danielhardman/), [Provenant, Inc](https://provenant.net)
- [Sam Smith](https://github.com/SmithSamuelM), [Prosapien](https://prosapien.com/)
- [Lance Byrd](https://github.com/2byrds), [GLEIF](https://gleif.org) and [RootsID](https://rootsid.com/)
- [Jonathan Rayback](https:/github.com/jrayback), [Future Forge Innovation LLC](https://futureforg.ing)
- [Kent Bull](https://github.com/kentbull), [Kent Bull](https://kentbull.com)

**Contributors:**

- [Markus Sabadello][msabadello],
  [Danube Tech](https://danubetech.com/) and [DIF](https://identity.foundation)
- [Kevin Griffin](https://github.com/m00sey), [GLEIF](https://gleif.org)
- [Charles Lanahan](https://www.linkedin.com/in/charles-lanahan-a8a68b16/)
- [Nuttawut Kongsuwan](https://github.com/nkongsuwan), [Finema](https://finema.co/)
- [Darrell O'Donnell](https://github.com/darrellodonnell), [Continuum Loop Inc.](https://www.continuumloop.com)
- [Henk van Cann](https:/github.com/henkvancann), [Blockchainbird](https://blockchainbird.org)
<!-- -->

**Participate:**

~ [GitHub repo](https://github.com/trustoverip/kswg-did-method-webs-specification)
~ [Commit history](https://github.com/trustoverip/kswg-did-method-webs-specification/commits/main)

[msabadello]: https://www.linkedin.com/in/markus-sabadello-353a0821/

## Introduction

::: informative Introduction

DID methods answer many questions. Two noteworthy ones are:

- How is information about DIDs (in the form of DID documents) published and discovered?
- How is the trustworthiness of this information evaluated?

The previously released `did:web` method merges these two questions, giving
one answer: _Information is published and secured using familiar web
mechanisms_. This has wonderful adoption benefits, because the processes
and tooling are familiar to millions of developers.

Unfortunately, this answer works better for the first question than the
second. The current web is simply not very trustworthy. Websites get
hacked. Sysadmins are sometimes malicious. DNS can be hijacked. X509
certs often prove less than clients wish. Browser validation checks are
imperfect. Different certificate authorities have different quality
standards. The processes that browser vendors use to pre-approve
certificate authorities in browsers are opaque and centralized. TLS is
susceptible to man-in-the-middle attacks on intranets with customized
certificate chains. Governance is weak and inconsistent...

Furthermore, familiar web mechanisms are almost always operated by
corporate IT staff. This makes them an awkward fit for the ideal of
decentralized autonomy — even if individuals can publish a DID on
corporate web servers, those individuals are at the mercy of IT personnel
for their security.

The `did:webs` method described in this spec separates these two questions
and answers them distinctively. _Information about DIDs_ is still
published on the web, but its _trustworthiness_ derives from mechanisms
entirely governed by individual DID controllers. This preserves most of the
delightful convenience of `did:web`, while drastically upgrading security
through authentic data that is end-verifiable.

Within the context of `did:webs` the term _decentralized trust_ includes
verifiability, confidentiality, and privacy, but excludes veracity of the
content. The latter is always a matter of (personal) evaluation of
available reputational data and verifiable credentials (VCs).

As a preview of syntax, see the below sample did:webs DID:

```text
did:webs:example.com%3A3000:users:alice:EKYGGh-FtAphGmSZbsuBs_t4qpsjYJ2ZqvMKluq9OxmP
│        │          │       │     │     │
│        │          │       │     │     └─ AID (KERI identifier)
│        │          │       │     └─────── Path component
│        │          │       └───────────── Path component
│        │          └───────────────────── Port (URL-encoded)
│        └──────────────────────────────── Host
└───────────────────────────────────────── Method
```

:::

## Status of This Memo

Information about the current status of this document, any errata,
and how to provide feedback on it, may be obtained at
[https://github.com/trustoverip/kswg-did-method-webs-specification](https://github.com/trustoverip/kswg-did-method-webs-specification).

## Copyright Notice

This specification is subject to the **OWF Contributor License Agreement 1.0 - Copyright**
available at
[https://www.openwebfoundation.org/the-agreements/the-owf-1-0-agreements-granted-claims/owf-contributor-license-agreement-1-0-copyright](https://www.openwebfoundation.org/the-agreements/the-owf-1-0-agreements-granted-claims/owf-contributor-license-agreement-1-0-copyright).

If source code is included in the specification, that code is subject to the
[Apache 2.0 license](https://www.apache.org/licenses/LICENSE-2.0.txt) unless otherwise marked. In the case of any conflict or
confusion between the OWF Contributor License and the designated source code license within this specification, the terms of the OWF Contributor License MUST apply.

These terms are inherited from the Technical Stack Working Group at the Trust over IP Foundation. [Working Group Charter](https://trustoverip.org/wp-content/uploads/TSWG-2-Charter-Revision.pdf).

## Terms of Use

These materials are made available under and are subject to the [OWF CLA 1.0 - Copyright & Patent license](https://www.openwebfoundation.org/the-agreements/the-owf-1-0-agreements-granted-claims/owf-contributor-license-agreement-1-0-copyright-and-patent). Any source code is made available under the [Apache 2.0 license](https://www.apache.org/licenses/LICENSE-2.0.txt).

THESE MATERIALS ARE PROVIDED “AS IS.” The Trust Over IP Foundation, established as the Joint Development Foundation Projects, LLC, Trust Over IP Foundation Series ("ToIP"), and its members and contributors (each of ToIP, its members and contributors, a "ToIP Party") expressly disclaim any warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to the materials. The entire risk as to implementing or otherwise using the materials is assumed by the implementer and user.
IN NO EVENT WILL ANY ToIP PARTY BE LIABLE TO ANY OTHER PARTY FOR LOST PROFITS OR ANY FORM OF INDIRECT, SPECIAL, INCIDENTAL, OR CONSEQUENTIAL DAMAGES OF ANY CHARACTER FROM ANY CAUSES OF ACTION OF ANY KIND WITH RESPECT TO THESE MATERIALS, ANY DELIVERABLE OR THE ToIP GOVERNING AGREEMENT, WHETHER BASED ON BREACH OF CONTRACT, TORT (INCLUDING NEGLIGENCE), OR OTHERWISE, AND WHETHER OR NOT THE OTHER PARTY HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

## Scope

This document specifies a [DID
Method](https://www.w3.org/TR/did-1.0/#methods),
`did:webs`, that is web-based but innovatively secure. Like its
interoperable cousin, [`did:web`](#W3C-DID-WEB), the
`did:webs` method uses traditional web infrastructure to publish DIDs and
make them discoverable. Unlike `did:web`, this method's trust is not rooted in
DNS, webmasters, X509, and certificate authorities. Instead, it uses [[ref:
KERI]] to provide a secure chain of cryptographic key events by those who
control the identifier including any of its delegators.

The `did:webs` method does not need blockchains to establish trust.
However, its use of KERI allows for arbitrary blockchains to be
referenced as an extra, optional publication mechanism. This offers a
potentital interoperability bridge from (or between) blockchain
ecosystems. Also, without directly supporting environments where the
web is not practical (e.g., IOT, Lo-Ra, Bluetooth, NFC), the method builds on a
foundation that can fully support those environments, making future interop of
identifiers between web and non-web a manageable step for users of `did:webs` identifiers.

All DID methods make tradeoffs. The ones in `did:webs` result in a method that
is cheap, easy to implement, and scalable. No exotic or unproven cryptography is
required. Deployment is straightforward. Cryptographic trust is strongly
decentralized and governance is transparent. Signing authority is scalable through
the support of delegated identifiers. Regulatory challenges around the issue of
blockchains vanish. Any tech community or legal jurisdiction can use it. However,
`did:webs` _does_ depend on the web for publication and discovery. This may
color its decentralization and privacy. For its security, it adds [[ref:
KERI]]. For users, the method also raises the bar of accountability,
thoughtfulness, and autonomy; this can be viewed as
either a drawback or a benefit (or both).

## Normative references

[The normative documents](#normative-section) are referred to in the text in such a way that some or all of their content constitutes requirements of this document. For dated references, only the edition cited applies. For undated references, the latest edition of the referenced document (including any amendments) applies.

- [ISO/IEC 7498-1](#IT7498):1994 Information technology — Open Systems Interconnection — Basic Reference Model: The Basic Model, including Requirement Levels (IETF [RFC-2119](#RFC2119)).

See [Bibliography - Normative Section](#normative-section)

## Terms and Definitions

For the purposes of this document, the following terms and definitions apply.

ISO and IEC maintain terminological databases for use in standardization at the following addresses:

 - ISO Online browsing platform: available at <https://www.iso.org/obp>
 - IEC Electropedia: available at <http://www.electropedia.org/>

:::
