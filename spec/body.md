## Core Characteristics

This section is normative.

### Method Name

1. The method name that identifies this DID method SHALL be: `webs`.
1. A DID that uses this method MUST begin with the following prefix: `did:webs:`.
1. Per the DID specification, this string MUST be lower case.
1. The remainder of the DID, after the prefix, MUST be the case-sensitive
   [[ref: method-specific identifier]] ([[ref: MSI]]) described
   [below](#method-specific-identifier).

::: informative Note on pronunciation
Note: when pronounced aloud, "webs" should become two syllables: the word
"web" and the letter "s" (which stands for "secure"). Separating the final
letter this way emphasizes that the method offers a security upgrade
surpassing the one HTTPS gives to HTTP.
:::

### Method-Specific Identifier

1. The `did:webs` [[ref: method-specific identifier]] MUST have two parts, a
   [[ref: host]] with an optional path (identical to `did:web`), plus a KERI
   [[ref: AID]] ([[ref: autonomic identifier]]) that is always the final
   component of the path.
1. The ABNF definition of a `did:webs` DID MUST be as follows:

```abnf
; did:webs DID structure
webs-did = "did:webs:" host [pct-encoded-colon port] *(":" path) ":" aid

; 'host' as defined in RFC 1035, RFC 1123, and RFC 2181
host = *( ALPHA / DIGIT / "-" / "." )
; Simplified representation; actual RFCs have more complex rules for domains
; and IP addresses. In actual implementations replace with a mature host
; parsing library.

; 'pct-encoded-colon' represents a percent-encoded colon
pct-encoded-colon = "%3A" / "%3a" ; Percent encoding for ':'

; 'port' number (simplified version)
port = 1*5(DIGIT)

; 'path' definition
path = 1*(ALPHA / DIGIT / "-" / "_" / "~" / "." / "/")

aid = said

; AID is a KERI SAID; SAID structure:
said = said-256 / said-512

; Base64URLSafe characters (RFC 4648, excluding padding)
base64urlsafe = ALPHA / DIGIT / "-" / "_"

; The complete SAID primitive MUST conform to CESR code table [2], CESR spec
; Section 11.4. The following currently defined digest codes produce 256-bit
; or 512-bit SAIDs of 44 or 88 characters total.

; 256-bit SAIDs: 44 characters total (1 char code + 43 Base64URLSafe)
one-char-code = "E" / "F" / "G" / "H" / "I"
said-256 = one-char-code 43base64urlsafe

; 512-bit SAIDs: 88 characters total (2 char code + 86 Base64URLSafe)
two-char-code = "0D" / "0E" / "0F" / "0G"
said-512 = two-char-code 86base64urlsafe
```

1. The [[ref: host]] MUST abide by the formal rules describing valid syntax
   found in [RFC 1035](#RFC1035), [RFC 1123](#RFC1123), and
   [RFC 2181](#RFC2181).
1. A port MAY be included and the colon MUST be percent encoded, like `%3a`,
   to prevent a conflict with paths.
1. Directories and subdirectories MAY optionally be included, delimited by
   colons rather than slashes.
1. The KERI AID is a unique [[ref: identifier]] and MUST be derived from the
   [[ref: inception event]] of a KERI identifier.

::: informative did:web compatibility
To be compatible with `did:web`, the AID is "just a path", the final (and
perhaps only) path element. The presence of the required AID as a path
element means that a `did:webs` always has a path, and so the "no path"
version of a `did:web` that implicitly uses the `.well-known` location is not
supported by `did:webs`. Any `did:webs` can be expressed as a `did:web` but
the inverse is not true--a `did:webs` must include an AID.
:::

### Target System(s)

::: informative hosting rules
The following rules indicate where the two `did:webs` artifacts, `did.json` and 
`keri.cesr`, are to be hosted on target systems and how they are to be resolved
for both `did:webs` and `did:web` resolution.
:::

1. As with `did:web`, `did:webs` MUST read data from whatever web server is
   referenced when the [[ref: host]] portion of one of its DIDs is resolved.
1. A `did:webs` DID MUST resolve to a [[ref: DID document]] using a simple
   text transformation to an HTTPS URL in the same way as a `did:web` DID.
1. A `did:web` DID and `did:webs` DID with the same 
   [[ref: method-specific-identifier]] SHOULD return the same DID document, 
   except for minor differences in the `id`, `controller`, and `alsoKnownAs` 
   top-level properties that pertain to the identifiers themselves.
1. As with `did:web`, the location of the `did:webs` [[ref: DID document]]
   MUST be determined by transforming the DID to an HTTPS URL as follows:
    1. MUST replace `did:webs` with `https://`
    1. MUST replace the "`:`"s in the method-specific identifier with path
       separators, "'/'"s
    1. MUST convert the optional port percent encoding ("`%3A`") to a colon
       if present.
    1. MUST append "`/did.json`" to the resulting string.
1. A GET on that URL MUST return the DID document.
1. The location of the [[ref: KERI event stream]] MUST be determined by
   transforming the previous URL as follows:
    1. MUST replace the trailing "`/did.json`" with "`/keri.cesr`".
    2. A GET on that URL MUST return the KERI event stream for the AID in
       the `did:webs` identifier.
    3. The KERI event stream MUST be [[ref: CESR]]-formatted, MUST have the
       media type of `application/cesr`, and the KERI events must be verifiable
       using the KERI rules.
1. The `did:web` version of the DIDs MUST be the same (minus the `s`) and
   point to the same `did.json` file.

#### Sample did:webs URLs

::: informative sample did:webs URLs on a target system

The below example `did:webs` DIDs and their corresponding DID
documents and KERI event stream URLs, based on the examples from the
[did:web specification](#W3C-DID-WEB), but with the sample AID `EKTh4PkRBiNWHQd263Eueu39gWmg7AfIfnEmNy6jinGR` added:

* `did:webs:w3c-ccg.github.io:EKTh4PkRBiNWHQd263Eueu39gWmg7AfIfnEmNy6jinGR`
    * The DID document URL would look like:
      `https://w3c-ccg.github.io/EKTh4PkRBiNWHQd263Eueu39gWmg7AfIfnEmNy6jinGR/did.json`
    * [[ref: KERI event stream]] URL would look like:
      `https://w3c-ccg.github.io/EKTh4PkRBiNWHQd263Eueu39gWmg7AfIfnEmNy6jinGR/keri.cesr`
* `did:webs:w3c-ccg.github.io:user:alice:EKTh4PkRBiNWHQd263Eueu39gWmg7AfIfnEmNy6jinGR`
    * The DID document URL would look like:
      `https://w3c-ccg.github.io/user/alice/EKTh4PkRBiNWHQd263Eueu39gWmg7AfIfnEmNy6jinGR/did.json`
    * [[ref: KERI event stream]] URL would look like:
      `https://w3c-ccg.github.io/user/alice/EKTh4PkRBiNWHQd263Eueu39gWmg7AfIfnEmNy6jinGR/keri.cesr`
* `did:webs:example.com%3A3000:user:alice:EKTh4PkRBiNWHQd263Eueu39gWmg7AfIfnEmNy6jinGR`
    * The DID document URL would look like:
      `https://example.com:3000/user/alice/EKTh4PkRBiNWHQd263Eueu39gWmg7AfIfnEmNy6jinGR/did.json`
    * [[ref: KERI event stream]] URL would look like:
      `https://example.com:3000/user/alice/EKTh4PkRBiNWHQd263Eueu39gWmg7AfIfnEmNy6jinGR/keri.cesr`

:::

#### KERI puts the "s" in `did:webs`

::: informative Target system and KERI verifiability
For more information, see the following sections in the implementors guide:

* [the set of KERI features needed](#the-set-of-keri-features-needed) to
  support `did:webs`

A target system cannot forge or tamper with data protected by KERI, and if
it deliberately serves an outdated copy, the duplicity is often detectable.
Thus, any given target system in isolation can be viewed by this method as a
dumb, untrusted server of content. It is the combination of target systems
and some KERI mechanisms, _together_, that constitutes this method's
verifiable data registry. In short, verifying the DID document by
processing the [[ref: KERI event stream]] using KERI puts the "s" of
"security" in `did:webs`.

:::

### AID controlled identifiers

1. [[ref: AID controlled identifiers]] MAY vary in how quickly they reflect
   the current identity information, DID document and [[ref: KERI event stream]]. 
   Notably, as defined in section
   [Stable Identifiers On An Unstable Web](#stable-identifiers-on-an-unstable-web), 
   the `id` property in the DID
   document will differ based on the web location of the DID document.
1. Different versions of the DID document and KERI event stream MAY reside
   in different locations depending on the replication capabilities of the
   controlling entity.
1. If the KERI event streams differ for `did:webs` DIDs with the same AID,
   the smaller KERI event stream MUST be a prefix of the larger KERI event
   stream (e.g., the only difference in the [[ref: KERI event streams]]
   being the extra events in one of the KERI event streams, not yet
   reflected in the other).
1. If the KERI event streams diverge from one other (e.g., one is not a
   subset of the other), both the KERI event streams and the DIDs MUST be
   considered invalid.
1. The verification of the KERI event stream SHOULD provide mechanisms for
   detecting the forking of the KERI event stream by using mechanisms such
   as KERI [[ref: witnesses]] and [[ref: watchers]].

::: informative AID and KERI event stream binding
Since an AID is a unique cryptographic identifier that is inseparably bound
to the [[ref: KERI event stream]] it is associated with any AIDs and any
`did:webs` DIDs that have the same AID component. It can be verifiably
proven that they have the same controller(s).
:::

### Handling Web Redirection

1. A `did:webs` DID MAY be a "stable" (long-lasting) identifier that can be
   put into documents such as verifiable credentials, to be useful for a
   very long time -- generations.
1. When a `did:webs` DID is updated for another location the following rules
   MUST apply:
    1. Its AID MUST not change.
    1. The same [[ref: KERI event stream]] MUST be used to verify the DID
       document, with the only change being the [[ref: designated aliases]]
       list reflecting the new location identifier.
    1. If a resolver can find a newly named DID that uses the same AID, and
       the KERI event stream verifies the DID, then the resolver MAY
       consider the resolution to be successful and should note it in the
       resolution metadata.

1. The following resolution paths that `did:webs` identifiers SHALL leverage
   to help in the face of resolution uncertainty include:
    1. The `did:webs` DID SHALL provide other [[ref: designated aliases]]
       DID(s) that are anchored to the [[ref: KERI event stream]].
    1. When a `did:webs` DID is permanently moved to some other location the
       resolver MAY redirect to any other `equivalentId` 
       [[ref: designated aliases]].
        1. The `id` in the DID document MUST be set to the new location.
        1. An `equivalentId` entry of the old location SHOULD remain for
           historical purposes and be anchored to the KERI event stream using
           [[ref: designated aliases]]. See section 
           [Use of `equivalentId`](#use-of-equivalentid) for more details.
        1. If possible, the controller of the DID MAY use web redirects to
           allow resolution of the old location of the DID to the new
           location.
    1. If the previously published location of a `did:webs` DID is not
       redirected, an entity trying to resolve the DID MAY be able to find
       the data for the DID somewhere else using just the AID.

::: informative Stable identifiers
The implementors guide contains more information about `did:webs` 
stable identifiers on an unstable web.
:::

### DID Method Operations

This section is normative.

::: informative mapping DID Method Operations to KERI events 
The four DID Method Operations, create, read, update, and deactivate conceptually map to the following events for a `did:webs` DID:
- Create: `did:webs` identifier creation maps to the combination of 
  - a KERI inception event and
  - initial issuance of a designated aliases ACDC with the initial host and path
    segments of the `did:webs` DID.
- Read: plain HTTP GET of the following resources. Similar to a KERI
  [[ref: OOBI]] resolution.
  - `did.json` document (derived from the `keri.cesr` stream)
  - `keri.cesr` stream 
    - includes:
      - KERI events (KEL) for the `did:webs` DID and any delegators
      - ACDC TEL events, issuance and revocation, for designated alias ACDCs
      - ACDCs for designated aliases
      - KERI Reply messages (Location Scheme, Endpoint Role Authorization)
- Update: involves changing values in the DID document. These might include:
  - **key change**: a KERI key rotation event changes the verificationMethods in a `did:webs` DID doc.
  - **alsoKnownAs** or **equivalentID** change: adding an alsoKnownAs or equivalentID identifier in a DID doc.
  - **service endpoint** change: [[ref: witness]] rotation, or other services may be added to the `service`
- Deactivate: rotating the underlying KERI identifier to null, or no next keys.
:::

#### Create

::: informative create operation
The **create** operation includes creation (inception) of the backing KERI 
[[ref: AID]] and issuance of the first designated aliases ACDC containing the 
host and path segment present in the `did:webs` DID.

Once these actions are complete then the `did.json` and `keri.cesr` data may be
generated and hosted at the URLs specified in the following rules.
:::

1. Creating a `did:webs` DID MUST follow these rules:
    1. MUST choose the web URL where the DID document for the DID will be
       published, excluding the last element that will be the AID, once
       defined.
    1. MUST create a KERI AID and add it as the last element of the web URL
       for the DID.
    1. MUST authorize the full `did:webs` identifier as a designated alias ACDC anchored to the KERI AID (see [Method-Specific Identifier](#method-specific-identifier)).
    1. If desired, a `did:web` form of the identifier MAY be authorized in the same manner.
    1. MUST add the appropriate KERI events to the AID's KERI logs that will
       correspond to properties of the DID document, such as verification
       methods and service endpoints.
    1. MUST derive the `did:webs` [[ref: DID document]] by processing the
       [[ref: KERI event stream]] according to section
       [DID Documents](#did-documents).
    1. For compatibility reasons, transformation of the derived `did:webs`
       DID document to the corresponding `did:web` DID document MUST be
       according to section 
       [Transformation to did:web DID Document](#transformation-to-didweb-did-document).
    1. MUST make the did:web DID document resource (`did.json`) and the
       [[ref: KERI event stream]] resource (`keri.cesr`) available at the
       selected location. See section [Target System(s)](#target-systems) for
       further details about the locations of these resources.

::: informative Publishing and hosting
Of course, the web server that serves the resources when asked might be a
simple file server (as implied above) or an active component that generates
them dynamically. Further, the publisher of the resources placed on the web
can use capabilities like [CDNs] to distribute the resources. How the
resources are posted at the required location is not defined by this spec;
complying implementations need not support any HTTP methods other than GET.

An active component might be used by the [[ref: controller]] of the DID to automate
the process of publishing and updating the DID document and [[ref: KERI
event stream]] resources.
:::

#### Read (Resolve)

1. Resolving a `did:webs` DID MUST follow these steps:
    1. MUST convert the `did:webs` DID back to HTTPS URLs as described in
       section [Target System(s)](#target-systems).
    1. MUST execute HTTP GET requests on both the URL for the DID document
       (ending in `/did.json`) and the URL for the [[ref: KERI event stream]] 
       (ending in `/keri.cesr`). To maintain compatibility with `did:web` resolvers, 
       the `did.json` MUST be formatted as a `did:web` DID document according to section 
       [Transformation to did:web](#transformation-to-didweb-did-document).
    1. MUST process the KERI event stream using 
       [KERI specification](#KSWG-KERI) rules
       to verify it, then derive the `did:webs` [[ref: DID document]] by
       processing the KERI event stream according to section [DID Documents](#did-documents).
    1. MUST transform the retrieved `did:web` DID document to the
       corresponding `did:webs` DID document according to section
       [Transformation to did:webs DID  Document](#transformation-to-didwebs-did-document).
    1. MUST verify that the derived `did:webs` DID document equals the
       transformed DID document. 
    1. MUST return the `did:webs` DID document.
    1. KERI-aware applications MAY use the KERI event stream to make use of
       additional capabilities enabled by the use of KERI.

::: informative Scope of KERI capabilities
Capabilities beyond the verification of the DID document, the KERI event
stream, and delegated identifiers are outside the scope of this
specification.
:::

#### Update

##### AID VersionID (Key) Update

The `did:webs` identifier itself, including the host, path, and AID cannot change. Updates 
include only rotations to new keys for the backing AID.

1. If the AID of the `did:webs` DID is updatable, as in transferable, 
   updates (key rotations) MUST be made to the AID by adding KERI key rotation 
  events to the [[ref: KERI event stream]].
1. Updates (rotations) to the KERI event stream that relate to the `did:webs` 
   DID MUST be reflected in the DID Document as soon as possible.
    1. If the `did:webs` DID files are statically hosted then they MUST be
       republished to the web server, overwriting the existing files.

Rotating keys MUST change the keys that show up in the DID Document and MUST NOT
change the identifier (AID).

> Note: the versionId query parameter is not a part of the identifier yet it may
> be used to request a point in time version of a DID document as of a given
> KERI key event sequence number.

#### Deactivate

1. To deactivate a `did:webs` DID, a [[ref: controller]] SHOULD execute a KERI event that
   has the effect of rotating the key(s) to null and continue to publish the
   DID document and KERI event stream.
    1. Once the deactivation events have been applied, the controller SHOULD
       regenerate the DID document from the [[ref: KERI event stream]] and
       republish both documents (`did.json` and `keri.cesr`) to the web
       server, overwriting the existing files.
    1. A controller SHOULD NOT make the DID document and 
       [[ref: KERI event stream]] resources unavailable at the location where they have been
       published.
        ::: informative Rationale for not removing DID files
        Removing the DID resources is considered to be a bad approach, as those resolving the DID
        will not be able to determine if the web service is offline or the
        DID has been deactivated.
        :::

## DID Documents

This section is normative.

1. `did:webs` DID documents MUST be generated or derived from the
   [[ref: KERI event stream]] of the corresponding AID.
    1. Processing the KERI event stream of the AID, the generation algorithm
       MUST read the AID [[ref: KEL]] and any anchored [[ref: TELs]] to
       produce the DID document, including any designated alias ACDCs.
2. `did:webs` DID documents MUST be pure JSON. They MAY be processed as
   JSON-LD by prepending an `@context` if consumers of the documents wish.
3. All hashes, cryptographic keys, and signatures MUST be represented as
   [[ref: CESR]] strings. This is an approach similar to
   [multibase](https://github.com/multiformats/multibase), making them
   self-describing and terse.

::: informative Understanding key state and KSN
To better understand the [[ref: cryptographically verifiable]] data structures used,
see the implementors guide description of the
[KERI event stream chain of custody](#KERI-event-stream-chain-of-custody).
To understand the KERI AID commands resulting in the
[[ref: KERI Event Stream]] and the corresponding `did:webs` DID document see
the original
[did:webs Reference Implementation getting started guide](#DIDWEBS-REF-IMPL).

In KERI the calculated values that result from processing the
[[ref: KERI event stream]] are referred to as the "current key state" and
expressed in the Key State Notice (KSN) record.
An example of a KERI KSN record can be seen here:

```json
{
    "v": "KERI10JSON000274_",
    "i": "EeS834LMlGVEOGR8WU3rzZ9M6HUv_vtF32pSXQXKP7jg",
    "s": "1",
    "t": "ksn",
    "p": "ESORkffLV3qHZljOcnijzhCyRT0aXM2XHGVoyd5ST-Iw",
    "d": "EtgNGVxYd6W0LViISr7RSn6ul8Yn92uyj2kiWzt51mHc",
    "f": "1",
    "dt": "2021-11-04T12:55:14.480038+00:00",
    "et": "ixn",
    "kt": "1",
    "k": ["DTH0PwWwsrcO_4zGe7bUR-LJX_ZGBTRsmP-ZeJ7fVg_4"],
    "nt": 1,
    "n": ["E6qpfz7HeczuU3dAd1O9gPPS6-h_dCxZGYhU8UaDY2pc"],
    "bt": "3",
    "b": [
      "BGKVzj4ve0VSd8z_AmvhLg4lqcC_9WYX90k03q-R_Ydo",
      "BuyRFMideczFZoapylLIyCjSdhtqVb31wZkRKvPfNqkw",
      "Bgoq68HCmYNUDgOz4Skvlu306o_NY-NrYuKAVhk3Zh9c"
    ],
    "c": [],
    "ee": {
      "s": "0",
      "d": "ESORkffLV3qHZljOcnijzhCyRT0aXM2XHGVoyd5ST-Iw",
      "br": [],
      "ba": []
    },
    "di": ""
}
```

Using this key state as reference, we can identify the fields from the
current key state that will translate to values in the DID document.
The following table lists the values from the example KSN and their
associated values in a DID document:

| Key State Field     | Definition                             | DID Document Value                  |
|:-------------------:|:--------------------------------------:|:-----------------------------------:|
| `i`                 | The AID value                          | The DID Subject and DID Controller  |
| `k`                 | Current set of public signing keys     | Verification Methods                |
| `kt`                | Current signing keys threshold         | Threshold in `ConditionalProof2022` |

In several cases above, the value from the key state is not enough by itself
to populate the DID document. The following sections detail the algorithm to
follow for each case.

:::

### DID Subject

This section is normative.

1. The value of the `id` property in the DID document MUST be the
   `did:webs` DID that is being created or resolved.
1. The value from the `i` field of the key state notice MUST be the value
   after the last `:` in the [[ref: method-specific identifier]] ([[ref: MSI]])
   of the `did:webs` DID, per section [Method-Specific Identifier](#method-specific-identifier).

```json
{
  "id": "did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M"
}
```

### DID Controller

This section is normative.

1. The value of the `controller` property MUST be a single string that is
   the same as the `id` (the DID Subject).

```json
{
  "controller": "did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M"
}
```

### Also Known As

This section is normative.

1. The `alsoKnownAs` property in the root of the DID document MAY contain
   any DID that has the same AID.
   ::: informative designated aliases reference
   See the [[ref: designated aliases]] section for information on how an AID
   anchors the `alsoKnownAs` identifiers to their [[ref: KERI event stream]].
   :::
   1. As long as the identifier is resolvable, a designated alias ACDC
      containing a given identifier MUST always be present in the `keri.cesr`
      stream in order for any identifier to be included in the `alsoKnownAs`
      section of a `did:webs` DID document.
      ::: informative transformation rules note
      Presence of designated alias ACDCs containing both `did:webs` and
      `did:web` identifiers are required to support the transformation rules
      between `did:webs` and `did:web` versions of a `did:webs` DID document
      while adhering to the security posture of KERI and ACDC.

      One potential way to implement this requirement is to ensure that
      resolving a given version of a `did:webs` DID document via the
      `versionId` parameter will return the DID document as of a given
      sequence number by analyzing the designated aliases ACDCs that were valid
      and unrevoked at that time.
      :::

1. The `did:webs` version of the DID document MAY include the `did:web`
   version of the DID as an `alsoKnownAs` identifier, provided that there is a valid, un-revoked designated aliases ACDC present in the `keri.cesr` stream.
1. The `did:web` version of the DID document MUST include the `did:webs`
   version of the DID as an `alsoKnownAs` identifier, meaning it must also be in a valid, un-revoked designated aliases ACDC present in the keri.cesr stream.
1. In order for the `did:webs` DID document to be valid, the `keri.cesr`
   stream MUST contain at least ONE designated aliases ACDC in which the DNS
   name and path for the `did:webs` identifier are committed to for both the
   `did:webs` and `did:web` versions of the identifier.
   ::: informative
   Committed to means placed in a designated aliases ACDC.

   This implies that the `did.json` for both the `did:webs` and `did:web`
   versions of a `did:webs` DID document will always contain a reciprocal
   link to one another that is also committed to by an event anchored into
   the KEL of the DID controller.

   A consumer of a DID document can only know that a given `did:web` DID is
   trustable and committed to by the controller of the AID supporting a
   `did:webs` DID only when that `did:web` DID is included in an un-revoked
   designated aliases ACDC.

   This protects against DID document malleability attacks where a malicious
   DID resolver host could inject fraudulent `did:web` DIDs into a DID
   document. As such, the consumer of a `did:webs` DID document should only
   trust `did:web` DIDs that are found in an un-revoked designated aliases
   ACDC present in the `keri.cesr` stream.
   :::
1. `did:webs` DIDs MUST provide the corresponding `did:keri` as an
   `alsoKnownAs` identifier.
1. The same AID MAY be associated with multiple `did:webs` DIDs, each with
   a different [[ref: host]] and/or path, but with the same AID.
1. `did:webs` DIDs MUST be listed in the Designated aliases attestation of the AID.
1. For each [[ref: AID controlled identifier]] DID defined above, an entry
   in the `alsoKnownAs` array in the DID document MUST be created.

::: informative example alsoKnownAs
For the example DID
`did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe`
the following `alsoKnownAs` entries could be created:

```json
{
  "alsoKnownAs": [
    "did:web:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
    "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
    "did:web:example.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
    "did:web:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
    "did:webs:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe"
  ]
}
```

:::

### Verification Methods

This section is normative.

Each verification method for a `did:webs` DID is generated from signing keys
located in the [[ref: KERI event stream]] of the controller of the
`did:webs` DID.

1. For each key listed in the array value of the `k` field of the KSN, a
   corresponding verification method MUST be generated in the DID document.
1. The `type` property in the verification method for each public key MUST
   be determined by the algorithm used to generate the public key.
1. The verification method types used MUST be registered in the
   [DID Specification Registries]
   (<https://www.w3.org/TR/did-extensions-properties/#verification-relationships>)
   and added to this specification.
1. The `id` property of the verification method MUST be a relative DID URL
   and use the KERI key [[ref: CESR]] value as the value of the fragment
   component, e.g., `"id": "#<identifier>"`.
1. The `controller` property of the verification method MUST be the value of
   the `id` property of the DID document.

   ::: informative controller and DID document id
   DID Core requires each verification method to have a `controller` property
   whose value is a valid DID, but does not require that value to equal the
   `id` of the DID document (e.g., delegation may use a different
   controller). This specification requires that for `did:webs` the
   `controller` of every verification method equals the document `id`, since
   all verification material is derived from the same AID's key state.
   :::

::: informative CESR and supported key types
KERI identifiers express public signing keys as Composable Event Streaming
Representation (CESR) encoded strings in the `k` field of establishment
events and the key state notice. CESR encoding encapsulates all the
information needed to determine the cryptographic algorithm used to
generate the key pair.

At the time of this writing, KERI currently supports public key generation
for Ed25519, Secp256k1 and Secp256r1 keys, and the protocol allows for
others to be added at any time.

For example, the key `DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr` in the
DID document for the AID
`ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe` becomes:

```json
  "verificationMethod": [
    {"id": "#DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr",
    "type": "JsonWebKey",
    "controller": "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe", 
    "publicKeyJwk": {
      "kid": "DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr", 
      "kty": "OKP", 
      "crv": "Ed25519", 
      "x": "evT4j6Yw3uHpwsw5NEmSR8-4x3S-BA-s6Thjd51oeOs"
      }
    }
  ]
```

:::

#### Ed25519

1. Ed25519 public keys MUST be converted to a verification method with a
   type of `JsonWebKey` and `publicKeyJwk` property whose value is generated
   by decoding the [[ref: CESR]] representation of the public key out of the
   KEL and into its binary form (minus the leading 'B' or 'D' CESR codes) and
   generating the corresponding representation of the key in JSON Web Key
   form.

For example, a KERI AID with only the following inception event in its KEL:

```json
{
  "v":"KERI10JSON00012b_",
  "t":"icp",
  "d":"ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
  "i":"ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
  "s":"0",
  "kt":"1",
  "k":["DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr"],
  // ...
}
```

would result in a DID document with the following verification methods array:

```json
"verificationMethod": [
  {
    "id": "#DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr", 
    "type": "JsonWebKey", 
    "controller": "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe", 
    "publicKeyJwk": {
      "kid": "DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr", 
      "kty": "OKP", 
      "crv": "Ed25519", 
      "x": "evT4j6Yw3uHpwsw5NEmSR8-4x3S-BA-s6Thjd51oeOs"
      }
  }
]
```

#### Secp256k1

1. Secp256k1 public keys MUST be converted to a verification method with a
   type of `JsonWebKey` and `publicKeyJwk` property whose value is generated
   by decoding the [[ref: CESR]] representation of the public key out of the
   KEL and into its binary form (minus the leading '1AAA' or '1AAB' CESR
   codes) and generating the corresponding representation of the key in
   JSON Web Key form.

For example, a KERI AID with only the following inception event in its KEL:

```json
{
  "v": "KERI10JSON0001ad_",
  "t": "icp",
  "d": "EDP1vHcw_wc4M__Fj53-cJaBnZZASd-aMTaSyWEQ-PC2",
  "i": "EDP1vHcw_wc4M__Fj53-cJaBnZZASd-aMTaSyWEQ-PC2",
  "s": "0",
  "kt": "1",
  "k": [
    "1AAAAmbFVu-Wf8NCd63B9V0zsy7EgB_ocX2_n_Nh1FCmgF0Y",
  ]
  // ...
}
```

would result in a DID document with the following verification methods array:

```json
  "verificationMethod": [
    {
      "id": "#1AAAAmbFVu-Wf8NCd63B9V0zsy7EgB_ocX2_n_Nh1FCmgF0Y",
      "type": "JsonWebKey",
      "controller": "did:webs:example.com:EDP1vHcw_wc4M__Fj53-cJaBnZZASd-aMTaSyWEQ-PC2",
      "publicKeyJwk": {
        "kid": "1AAAAmbFVu-Wf8NCd63B9V0zsy7EgB_ocX2_n_Nh1FCmgF0Y",
        "kty": "EC",
        "crv": "secp256k1",
        "x": "ZsVW75Z_w0J3rcH1XTOzLsSAH-hxfb-Q82HUUKaAXRg",
        "y": "Lu6Uw785U3K05D-NPNoUInHPNUz9cGqWwjKjm5KL8FI"
      }
    }
  ]
```
#### Secp256r1

1. Secp256r1 public keys MUST be converted to a verification method with a
   type of `JsonWebKey` and `publicKeyJwk` property whose value is generated
   by decoding the [[ref: CESR]] representation of the public key out of the
   KEL and into its binary form (minus the leading '1AAI' or '1AAJ' CESR
   codes) and generating the corresponding representation of the key in
   JSON Web Key form.

For example, a KERI AID with only the following inception event in its KEL:

```json
{
  "v": "KERI10JSON0001ad_",
  "t": "icp",
  "d": "EDP1vHcw_wc4M__Fj53-cJaBnZZASd-aMTaSyWEQ-PC2",
  "i": "EDP1vHcw_wc4M__Fj53-cJaBnZZASd-aMTaSyWEQ-PC2",
  "s": "0",
  "kt": "1",
  "k": [
    "1AAIAmbFVu-Wf8NCd63B9V0zsy7EgB_ocX2_n_Nh1FCmgF0Y",
  ]
  // ...
}
```

would result in a DID document with the following verification methods array:

```json
  "verificationMethod": [
    {
      "id": "#1AAIAmbFVu-Wf8NCd63B9V0zsy7EgB_ocX2_n_Nh1FCmgF0Y",
      "type": "JsonWebKey",
      "controller": "did:webs:example.com:EDP1vHcw_wc4M__Fj53-cJaBnZZASd-aMTaSyWEQ-PC2",
      "publicKeyJwk": {
        "kid": "1AAIAmbFVu-Wf8NCd63B9V0zsy7EgB_ocX2_n_Nh1FCmgF0Y",
        "kty": "EC",
        "crv": "secp256r1",
        "x": "ZsVW75Z_w0J3rcH1XTOzLsSAH-hxfb-Q82HUUKaAXRg",
        "y": "Lu6Uw785U3K05D-NPNoUInHPNUz9cGqWwjKjm5KL8FI"
      }
    }
  ]
```
#### Thresholds

1. If the current signing keys threshold (the value of the `kt` field) is a
   string containing a number that is greater than 1, or if it is an array
   containing fractionally weighted thresholds, then in addition to the
   verification methods generated according to the rules in the previous
   sections, another 
   [verification method](https://w3c-ccg.github.io/verifiable-conditions/)
   with a type of `ConditionalProof2022` MUST be generated in the DID document.
   This verification method type is defined.
    1. It MUST be constructed according to the following rules:
        1. The `id` property of the verification method MUST be a relative
           DID URL and use the AID as the value of the fragment component,
           e.g., `"id": "#<aid>"`.
        1. The `controller` property of the verification method MUST be the
           value of the `id` property of the DID document.
        1. If the value of the `kt` field is a string containing a number
           that is greater than 1 then the following rules MUST be applied:
            1. The `threshold` property of the verification method MUST be
               the integer value of the `kt` field in the current key state.
            1. The `conditionThreshold` property of the verification method
               MUST contain an array. For each key listed in the array value
               of the `k` field in the key state:
                1. The relative DID URL corresponding to the key MUST be added
                   to the array value of the `conditionThreshold` property.
        1. If the value of the `kt` field is an array containing fractionally
           weighted thresholds then the following rules MUST be applied:
            1. The `threshold` property of the verification method MUST be
               the lowest common denominator (LCD) of all the fractions in
               the `kt` array.
            1. The `conditionWeightedThreshold` property of the verification
               method MUST contain an array. For each key listed in the array
               value of the `k` field in the key state, and for each
               corresponding fraction listed in the array value of the `kt`
               field:
                1. A JSON object MUST be added to the array value of the
                   `conditionWeightedThreshold` property.
                1. The JSON object MUST contain a property `condition` whose
                   value is the relative DID URL corresponding to the key.
                1. The JSON object MUST contain a property `weight` whose
                   value is the numerator of the fraction after it has been
                   expanded over the lowest common denominator (LCD) of all
                   the fractions.

        For example, a KERI AID with only the following inception event in
        its KEL, and with a `kt` value greater than 1:

        ```json
        {
          "v": "KERI10JSON0001b7_",
          "t": "icp",
          "d": "Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
          "i": "Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
          "s": "0",
          "kt": "2",  // Signing Threshold
          "k": [
            "1AAAAg299p5IMvuw71HW_TlbzGq5cVOQ7bRbeDuhheF-DPYk",  // Secp256k1 Key
            "DA-vW9ynSkvOWv5e7idtikLANdS6pGO2IHJy7v0rypvE",      // Ed25519 Key
            "DLWJrsKIHrrn1Q1jy2oEi8Bmv6aEcwuyIqgngVf2nNwu"       // Ed25519 Key
          ],
        }
        ```

        results in a DID document with the following verification methods array:

        ```json
        {
          "verificationMethod": [
            {
              "id": "#Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
              "type": "ConditionalProof2022",
              "controller": "did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
              "threshold": 2,
              "conditionThreshold": [
                "#1AAAAg299p5IMvuw71HW_TlbzGq5cVOQ7bRbeDuhheF-DPYk",
                "#DA-vW9ynSkvOWv5e7idtikLANdS6pGO2IHJy7v0rypvE",
                "#DLWJrsKIHrrn1Q1jy2oEi8Bmv6aEcwuyIqgngVf2nNwu"
              ]
            },
            {
              "id": "#1AAAAg299p5IMvuw71HW_TlbzGq5cVOQ7bRbeDuhheF-DPYk",
              "type": "JsonWebKey",
              "controller": "did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
              "publicKeyJwk": {
                "kid": "1AAAAg299p5IMvuw71HW_TlbzGq5cVOQ7bRbeDuhheF-DPYk",
                "kty": "EC",
                "crv": "secp256k1",
                "x": "NtngWpJUr-rlNNbs0u-Aa8e16OwSJu6UiFf0Rdo1oJ4",
                "y": "qN1jKupJlFsPFc1UkWinqljv4YE0mq_Ickwnjgasvmo"
              }
            },
            {
              "id": "#DA-vW9ynSkvOWv5e7idtikLANdS6pGO2IHJy7v0rypvE",
              "type": "JsonWebKey",
              "controller": "did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
              "publicKeyJwk": {
                "kid": "DA-vW9ynSkvOWv5e7idtikLANdS6pGO2IHJy7v0rypvE",
                "kty": "OKP",
                "crv": "Ed25519",
                "x": "A-vW9ynSkvOWv5e7idtikLANdS6pGO2IHJy7v0rypvE"
              }
            },
            {
              "id": "#DLWJrsKIHrrn1Q1jy2oEi8Bmv6aEcwuyIqgngVf2nNwu",
              "type": "JsonWebKey",
              "controller": "did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
              "publicKeyJwk": {
                "kid": "DLWJrsKIHrrn1Q1jy2oEi8Bmv6aEcwuyIqgngVf2nNwu",
                "kty": "OKP",
                "crv": "Ed25519",
                "x": "LWJrsKIHrrn1Q1jy2oEi8Bmv6aEcwuyIqgngVf2nNws"
              }
            }
          ]
        }
        ```

        For example, a KERI AID with only the following inception event in
        its KEL, and a `kt` containing fractionally weighted thresholds:

        ```json
        {
          "v": "KERI10JSON0001b7_",
          "t": "icp",
          "d": "Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
          "i": "Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
          "s": "0",
          "kt": ["1/2", "1/3", "1/4"],  // Signing Threshold
          "k": [
            "1AAAAg299p5IMvuw71HW_TlbzGq5cVOQ7bRbeDuhheF-DPYk",  // Secp256k1 Key
            "DA-vW9ynSkvOWv5e7idtikLANdS6pGO2IHJy7v0rypvE",      // Ed25519 Key
            "DLWJrsKIHrrn1Q1jy2oEi8Bmv6aEcwuyIqgngVf2nNwu"       // Ed25519 Key
          ],
        }
        ```

        would result in a DID document with the following verification methods array:

        ```json
        {
          "verificationMethod": [
            {
              "id": "#Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
              "type": "ConditionalProof2022",
              "controller": "did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
              "threshold": 12,
              "conditionWeightedThreshold": [
                {
                  "condition": "#1AAAAg299p5IMvuw71HW_TlbzGq5cVOQ7bRbeDuhheF-DPYk",
                  "weight": 6
                },
                {
                  "condition": "#DA-vW9ynSkvOWv5e7idtikLANdS6pGO2IHJy7v0rypvE",
                  "weight": 4
                },
                {
                  "condition": "#DLWJrsKIHrrn1Q1jy2oEi8Bmv6aEcwuyIqgngVf2nNwu",
                  "weight": 3
                }
              ]
            },
            {
              "id": "#1AAAAg299p5IMvuw71HW_TlbzGq5cVOQ7bRbeDuhheF-DPYk",
              "type": "EcdsaSecp256k1VerificationKey2019",
              "controller": "did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
              "publicKeyJwk": {
                "crv": "secp256k1",
                "x": "NtngWpJUr-rlNNbs0u-Aa8e16OwSJu6UiFf0Rdo1oJ4",
                "y": "qN1jKupJlFsPFc1UkWinqljv4YE0mq_Ickwnjgasvmo",
                "kty": "EC",
                "kid": "WjKgJV7VRw3hmgU6--4v15c0Aewbcvat1BsRFTIqa5Q"
              }
            },
            {
              "id": "#DA-vW9ynSkvOWv5e7idtikLANdS6pGO2IHJy7v0rypvE",
              "type": "Ed25519VerificationKey2020",
              "controller": "did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
              "publicKeyMultibase": "zH3C2AVvLMv6gmMNam3uVAjZpfkcJCwDwnZn6z3wXmqPV"
            },
            {
              "id": "#DLWJrsKIHrrn1Q1jy2oEi8Bmv6aEcwuyIqgngVf2nNwu",
              "type": "Ed25519VerificationKey2020",
              "controller": "did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
              "publicKeyMultibase": "zDqYpw38nznAUJeeFdhKBQutRKpyDXeXxxi1HjYUQXLas"
            }
          ]
        }
        ```

### Verification Relationships

This section is normative.

`did:webs` commits to same keys for both authentication and assertion, a
design facilitated by being built upon KERI. This section defines how the
dual use of keys for `authentication` and `assertionMethod` is reflected
normatively in verification relationships. A `did:webs` DID document MAY
include any of these, or other properties, to express a specific
verification relationship. Both the `authentication` and `assertionMethod`
properties are optional though if included MUST follow the rules stated in
this section. When verification relationships are present in a `did:webs`
DID document, each committed signing key for a given `did:webs` DID MUST
show up as both an `authentication` and `assertionMethod` verification
relationship in the DID document.

1. If the value of `kt` == 1 then the following rules MUST be applied:
    1. For each public key in `k` and its corresponding verification method,
       two verification relationships MUST be generated in the DID document.
       One verification relationship of type `authentication` and one of type
       `assertionMethod`.
        1. The `authentication` verification relationship SHALL define that
           the DID controller can authenticate using each key.
        1. The `assertionMethod` verification relationship SHALL define that
           the DID controller can express claims using each key.
1. If the value of `kt` > 1 or if the value of `kt` is an array containing
   fractionally weighted thresholds then the following rules MUST be applied:
    1. For the verification method of type `ConditionalProof2022` (see section
       [Thresholds](#thresholds)), two verification relationships MUST be
       generated in the DID document. One of type `authentication` and one of
       type `assertionMethod`.
        1. The `authentication` verification relationship SHALL define that
           the DID controller can authenticate using a combination of
           multiple keys above the threshold.
        1. The `assertionMethod` verification relationship SHALL define that
           the DID controller can express claims using a combination of
           multiple keys above the threshold.
1. References to verification methods in the DID document MUST use the
   relative form of the identifier, e.g., `"authentication": ["#<identifier>"]`.

For example, a KERI AID with only the following inception event in its KEL:

```json
{
  "v":"KERI10JSON00012b_",
  "t":"icp",
  "d":"ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
  "i":"ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
  "s":"0",
  "kt":"1",
  "k":["DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr"],
  // ...
}
```

would result in a DID document with the following verification methods array and verification relationships:

```json
"verificationMethod": [
  {
    "id": "#DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr", 
    "type": "JsonWebKey", 
    "controller": "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe", 
    "publicKeyJwk": {
      "kid": "DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr", 
      "kty": "OKP", 
      "crv": "Ed25519", 
      "x": "evT4j6Yw3uHpwsw5NEmSR8-4x3S-BA-s6Thjd51oeOs"
      }
  }
],
"authentication": [
  "#DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr"
],
"assertionMethod": [
  "#DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr"
]
```

::: informative Use of private keys and key agreement
Private keys of a KERI AID can be used to sign a variety of data. This
includes but is not limited to logging into a website, challenge-response
exchanges, credential issuances, etc.

For more information, see the [key agreement](#key-agreement) and
[other key commitments](#other-key-commitments) section in the Implementors
Guide.
:::

### Service Endpoints

This section is normative.

1. `did:webs` DIDs MUST support service endpoints, including types declared
   in the DID Specification Registries, such as
   [DIDCommMessaging](https://www.w3.org/TR/did-extensions-properties/#didcommmessaging).

::: informative Service endpoint mapping and metadata
For additional details about the mapping between KERI events and the Service
Endpoints in the DID Document, such as 
- witness,
- mailbox,
- delegator OOBI, and 
- agent service endpoints, 

see [Service Endpoint KERI events](#service-endpoint-event-details).

It is important to note that DID document service endpoints are different
than the KERI service endpoints detailed in
[KERI Service Endpoints as DID Document Metadata]
(#keri-service-endpoints-as-did-document-metadata).
:::

#### KERI Service Endpoints as DID Document Metadata

1. `did:webs` endpoints MUST be specified using the two data sets KERI uses
   to define service endpoints: Location Schemes and Endpoint Role
   Authorizations.
    1. Both MUST be expressed in KERI `rpy` events.
    1. For URL scheme endpoints that an AID has exposed, `did:webs` DIDs
       MUST use Location Schemes URLs.
    1. For endpoints that relate a role of one AID to another, `did:webs`
       DIDs MUST use KERI Endpoint Role Authorizations.

    For example, the following `rpy` method declares that the AID
    `EIDJUg2eR8YGZssffpuqQyiXcRVz2_Gw_fcAVWpUMie1` exposes the URL
    `http://localhost:3902` for scheme `http`:

    ```json
    {
        // ...
        "t": "rpy",
        "r": "/loc/scheme",
        "a": {
          "eid": "EIDJUg2eR8YGZssffpuqQyiXcRVz2_Gw_fcAVWpUMie1",
          "scheme": "http",
          "url": "http://127.0.0.1:3901/"
        }
      }
    ```

    For example, the AID listed in `cid` is the source of the authorization,
    the `role` is the role and the AID listed in the `eid` field is the
    target of the authorization. So in this example
    `EOGL1KGpOnRaZDIB11uZDCkhHs52_MtMXHd7EqUqwtA3` is being authorized as an
    Agent for `EIDJUg2eR8YGZssffpuqQyiXcRVz2_Gw_fcAVWpUMie1`.

    ```json
    {
        // ...
        "t": "rpy",
        "r": "/end/role/add",
        "a": {
          "cid": "EIDJUg2eR8YGZssffpuqQyiXcRVz2_Gw_fcAVWpUMie1",
          "role": "agent",
          "eid": "EOGL1KGpOnRaZDIB11uZDCkhHs52_MtMXHd7EqUqwtA3"
        }
    }
    ```

1. KERI service endpoints roles beyond `witness` SHOULD be defined using
   Location Scheme and Endpoint Authorization records in KERI. See the
   [KERI specification](<https://trustoverip.github.io/kswg-keri-specification/#oobi-url-iurl>)
   for more information about KERI roles.

::: informative BADA-RUN and service endpoints
In KERI, service endpoints are defined by 2 sets of signed data using Best
Available Data - Read, Update, Nullify ([[ref: BADA-RUN]]) rules for data
processing. The protocol ensures that all data is signed in transport and
at rest and versioned to ensure only the latest signed data is available.
:::

### Transformation to `did:web` DID Document

This section is normative.

When transforming a `did:webs` DID document to the corresponding `did:web` DID document
the following rules determine what MUST change to make a valid, transformed `did:web`
version of the `did:webs` DID document. 

::: informative properties affected by transformation
This transformation primarily affects 
the `id` and `controller` properties, and, when authorized with a designated aliases
ACDC, the `alsoKnownAs` properties.
:::

1. Transformation of the `did:webs` form of the DID Document to a `did:web` DID document
   MUST do the following:
    1. The top-level `id` and `controller` property values of the DID document 
       MUST replace the `did:webs` prefix string with the `did:web` prefix.
    1. If authorized by a designated alias ACDC, the `did:webs` identifier contained in the top-level `alsonKnownAs` property MUST be replaced with the corresponding, authorized, `did:web` identifier (the same `did:web` identifier formerly contained in the `id` and `controller` fields.)
    1. All other content of the DID document MUST not be modified.

    For example, this transformation is used during the [Create](#create) DID
    method operation, given the following `did:webs` DID document:

    ```json
    {
        "id": "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
        "verificationMethod": [
            {
                "id": "#DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr",
                "type": "JsonWebKey",
                "controller": "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
                "publicKeyJwk": {
                    "kid": "DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr",
                    "kty": "OKP",
                    "crv": "Ed25519",
                    "x": "evT4j6Yw3uHpwsw5NEmSR8-4x3S-BA-s6Thjd51oeOs"
                }
            }
        ],
        "service": [],
        "alsoKnownAs": [
            "did:web:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:web:example.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:web:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:webs:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe"
        ]
    }
    ```

    the result of the transformation algorithm is the following `did:web` DID document:

    ```json
    {
      "id": "did:web:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
      "verificationMethod": [
          {
              "id": "#DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr",
              "type": "JsonWebKey",
              "controller": "did:web:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
              "publicKeyJwk": {
                  "kid": "DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr",
                  "kty": "OKP",
                  "crv": "Ed25519",
                  "x": "evT4j6Yw3uHpwsw5NEmSR8-4x3S-BA-s6Thjd51oeOs"
              }
          }
      ],
      "service": [],
      "alsoKnownAs": [
          "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
          "did:web:example.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
          "did:web:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
          "did:webs:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe"
      ]
    }
    ```

### Transformation to `did:webs` DID Document

This section is normative.

When transforming a `did:web` DID document to the corresponding `did:webs` 
DID document the following rules determine what MUST change to make a valid, 
transformed `did:webs` version of the `did:web` DID document.

::: informative properties affected by transformation
This transformation primarily affects
the `id` and `controller` properties, and, when authorized with a designated 
aliases ACDC, the `alsoKnownAs` properties.
:::

1. Transformation of the `did:web` DID document to a `did:webs` DID document 
   MUST do the following:
    1. The top-level `id` and `controller` property values of the DID document
       MUST replace the `did:web` prefix string with the `did:webs` prefix.
    1. The `did:web` identifier contained in the top-level `alsonKnownAs` property MUST be replaced with the corresponding `did:webs` identifier (the same `did:webs` identifier formerly contained in the `id` and `controller` fields.)
    1. All other content of the DID document MUST not be modified.
1. A `did:webs` resolver MUST use this transformation during the
   [Read (Resolve)](#read-resolve) DID method operation.

    For example, given the following `did:web` DID document:

    ```json
    {
      "id": "did:web:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
      "verificationMethod": [
          {
              "id": "#DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr",
              "type": "JsonWebKey",
              "controller": "did:web:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
              "publicKeyJwk": {
                  "kid": "DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr",
                  "kty": "OKP",
                  "crv": "Ed25519",
                  "x": "evT4j6Yw3uHpwsw5NEmSR8-4x3S-BA-s6Thjd51oeOs"
              }
          }
      ],
      "service": [],
      "alsoKnownAs": [
          "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
          "did:web:example.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
          "did:web:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
          "did:webs:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe"
      ]
    }
    ```

    the result of the transformation algorithm is the following `did:webs`
    DID document:

    ```json
    {
        "id": "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
        "verificationMethod": [
            {
                "id": "#DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr",
                "type": "JsonWebKey",
                "controller": "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
                "publicKeyJwk": {
                    "kid": "DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr",
                    "kty": "OKP",
                    "crv": "Ed25519",
                    "x": "evT4j6Yw3uHpwsw5NEmSR8-4x3S-BA-s6Thjd51oeOs"
                }
            }
        ],
        "service": [],
        "alsoKnownAs": [
            "did:web:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:web:example.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:web:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:webs:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe"
        ]
    }
    ```

### Full Example

::: informative Full Example

To walk through a real-world example, please see the GETTING STARTED guide in
[the did:webs Reference Implementation](#DIDWEBS-REF-IMPL) as it walks users through many
did:webs related tasks (and associated KERI commands) to demonstrate how they
work together.

The following blocks contain fully annotated examples of a KERI AID with
two events, an [[ref: inception event]] and an [[ref: interaction event]].

* The Inception event designates some [[ref: witnesses]] in the `b` field.
* The Inception event designates multiple public signing keys in the `k` field.
* The Inception event designates multiple rotation keys in the `n` field.
* The Interaction event cryptographically anchors data associated with the
  SAID `EoLNCdag8PlHpsIwzbwe7uVNcPE1mTr-e1o9nCIDPWgM`.
* The reply `rpy` events specify a Witness endpoint, etc.

Below, we show the KERI Event Stream that will be associated with the
resulting generated DID document.

```json
{
  "v": "KERI10JSON000159_",
  "t": "icp",
  "d": "EEOqE46OOSl1k1JO3ggQTGuQR3nnWE8bYjOPnJ53m8CP",
  "i": "EEOqE46OOSl1k1JO3ggQTGuQR3nnWE8bYjOPnJ53m8CP", // controller AID
  "s": "0",
  "kt": "1", // Signing Threshold
  "k": [
    "DJg7AQUSKAEo-Pkgj4tVF7L0-FqJt0QFxFh5878AcZv6" // Ed25519 Key
  ],
  "nt": "1",
  "n": [
    "EIzv5mDvyc-ftAyn06LyO0HvzfG7dJqx1zvYWGGIGiYp"
  ],
  "bt": "1",
  "b": [
    "BJqHtDoLT_K_XyOgr2ejBOqD9276TYMTg2EEqWKs-V0q"
  ],
  "c": [],
  "a": []
}
// series of events establishing a witness service
{
    "v": "KERI10JSON00013a_",
    "t": "ixn",
    "d": "EMllQ2kDZp9OnErTfJkzqbZN4hoH-rATBvVyASczJFIN",
    "i": "EEOqE46OOSl1k1JO3ggQTGuQR3nnWE8bYjOPnJ53m8CP",
    "s": "1",
    "p": "EEOqE46OOSl1k1JO3ggQTGuQR3nnWE8bYjOPnJ53m8CP",
    "a": [
        {
            "i": "EBXOxFQrvOxBPb7DiRaTlZNLvX5RbL5cO05MdA35Z0vl",
            "s": "0",
            "d": "EBXOxFQrvOxBPb7DiRaTlZNLvX5RbL5cO05MdA35Z0vl"
        }
    ]
}
{
    "v": "KERI10JSON00013a_",
    "t": "ixn",
    "d": "EGZKcIKf9euYtsGJI5RxWoL6sprJS8Rt6Em97X85tJPq",
    "i": "EEOqE46OOSl1k1JO3ggQTGuQR3nnWE8bYjOPnJ53m8CP",
    "s": "2",
    "p": "EMllQ2kDZp9OnErTfJkzqbZN4hoH-rATBvVyASczJFIN",
    "a": [
        {
            "i": "EA7VoZA6B9hmhmJAuWUU4lNxRBIQ3sv6JZXGeLnJCRGZ",
            "s": "0",
            "d": "EBCVtYB15CiJg79EciGiAkQaU3L-4oCx7qcPqUsQGZqr"
        }
    ]
}
{
    "v": "KERI10JSON000109_",
    "t": "rpy",
    "d": "EAVexvYonCqxXNNQ8208SLo3Afcg9W7d7eiNJ2FC8tMe",
    "dt": "2025-04-16T16:50:26.283032-07:00",
    "r": "/loc/scheme",
    "a": {
        "eid": "BJqHtDoLT_K_XyOgr2ejBOqD9276TYMTg2EEqWKs-V0q",
        "scheme": "https",
        "url": "https://wit1.did-webs-service:5641/"
    }
}
{
    "v": "KERI10JSON000105_",
    "t": "rpy",
    "d": "ENAFmV935lXzYQfDKwgjU08o-AC5ZfqhGR2dtgka1kvq",
    "dt": "2025-04-16T16:50:26.283032-07:00",
    "r": "/loc/scheme",
    "a": {
        "eid": "BJqHtDoLT_K_XyOgr2ejBOqD9276TYMTg2EEqWKs-V0q",
        "scheme": "tcp",
        "url": "tcp://wit1.did-webs-service:5631/"
    }
}
{
    "v": "KERI10JSON000116_",
    "t": "rpy",
    "d": "EIUHDBQmdOf7B0TwHqREnB-_gGYa2b9exOcLJMylrp6P",
    "dt": "2025-04-16T16:50:26.283032-07:00",
    "r": "/end/role/add",
    "a": {
        "cid": "BJqHtDoLT_K_XyOgr2ejBOqD9276TYMTg2EEqWKs-V0q",
        "role": "controller",
        "eid": "BJqHtDoLT_K_XyOgr2ejBOqD9276TYMTg2EEqWKs-V0q"
    }
}
// issuance of designated alias ACDC authorizing host information for did:webs and did:web
{
    "v": "KERI10JSON0000ff_",
    "t": "vcp",
    "d": "EBXOxFQrvOxBPb7DiRaTlZNLvX5RbL5cO05MdA35Z0vl",
    "i": "EBXOxFQrvOxBPb7DiRaTlZNLvX5RbL5cO05MdA35Z0vl",
    "ii": "EEOqE46OOSl1k1JO3ggQTGuQR3nnWE8bYjOPnJ53m8CP",
    "s": "0",
    "c": [
     "NB"
    ],
    "bt": "0",
    "b": [],
    "n": "0AAuhv_GQrchdHyeZGXUMFw1"
}
{
    "v": "KERI10JSON0000ed_",
    "t": "iss",
    "d": "EBCVtYB15CiJg79EciGiAkQaU3L-4oCx7qcPqUsQGZqr",
    "i": "EA7VoZA6B9hmhmJAuWUU4lNxRBIQ3sv6JZXGeLnJCRGZ",
    "s": "0",
    "ri": "EBXOxFQrvOxBPb7DiRaTlZNLvX5RbL5cO05MdA35Z0vl",
    "dt": "2026-02-06T21:44:36.788115+00:00"
}
{
    "v": "ACDC10JSON000574_",
    "d": "EA7VoZA6B9hmhmJAuWUU4lNxRBIQ3sv6JZXGeLnJCRGZ",
    "i": "EEOqE46OOSl1k1JO3ggQTGuQR3nnWE8bYjOPnJ53m8CP",
    "ri": "EBXOxFQrvOxBPb7DiRaTlZNLvX5RbL5cO05MdA35Z0vl",
    "s": "EN6Oh5XSD5_q2Hgu-aqpdfbVepdpYpFlgz6zvJL5b_r5",
    "a": {
        "d": "ELt92UqnBvlk8UOWpK2uUWlXEohe6j_JEVP4zvRY3HB-",
        "dt": "2026-02-06T21:44:36.788115+00:00",
        "ids": [
            "did:web:did-webs-service%3a7702:EEOqE46OOSl1k1JO3ggQTGuQR3nnWE8bYjOPnJ53m8CP",
            "did:webs:did-webs-service%3a7702:EEOqE46OOSl1k1JO3ggQTGuQR3nnWE8bYjOPnJ53m8CP"
        ]
    },
    "r": {
    "d": "EEVTx0jLLZDQq8a5bXrXgVP0JDP7j8iDym9Avfo8luLw",
    "aliasDesignation": {
      "l": "The issuer of this ACDC designates the identifiers in the ids field as the only allowed namespaced aliases of the issuer's AID."
    },
    "usageDisclaimer": {
      "l": "This attestation only asserts designated aliases of the controller of the AID, that the AID controlled namespaced alias has been designated by the controller. It does not assert that the controller of this AID has control over the infrastructure or anything else related to the namespace other than the included AID."
    },
    "issuanceDisclaimer": {
      "l": "All information in a valid and non-revoked alias designation assertion is accurate as of the date specified."
    },
    "termsOfUse": {
      "l": "Designated aliases of the AID must only be used in a manner consistent with the expressed intent of the AID controller."
    }
    }
}

```

Resulting DID document:

```json
{
  "id": "did:webs:did-webs-service%3A7702:EEOqE46OOSl1k1JO3ggQTGuQR3nnWE8bYjOPnJ53m8CP",
  "verificationMethod": [
    {
      "id": "#DJg7AQUSKAEo-Pkgj4tVF7L0-FqJt0QFxFh5878AcZv6",
      "type": "JsonWebKey",
      "controller": "did:webs:did-webs-service%3A7702:EEOqE46OOSl1k1JO3ggQTGuQR3nnWE8bYjOPnJ53m8CP",
      "publicKeyJwk": {
        "kid": "DJg7AQUSKAEo-Pkgj4tVF7L0-FqJt0QFxFh5878AcZv6",
        "kty": "OKP",
        "crv": "Ed25519",
        "x": "mDsBBRIoASj4-SCPi1UXsvT4Wom3RAXEWHnzvwBxm_o"
      }
    }
  ],
  "service": [
    {
      "id": "#BJqHtDoLT_K_XyOgr2ejBOqD9276TYMTg2EEqWKs-V0q/witness",
      "type": "witness",
      "serviceEndpoint": {
        "https": "https://wit1.did-webs-service:5641/",
        "tcp": "tcp://wit1.did-webs-service:5631/"
      }
    }
  ],
  "alsoKnownAs": [
    "did:web:did-webs-service%3a7702:EEOqE46OOSl1k1JO3ggQTGuQR3nnWE8bYjOPnJ53m8CP",
    "did:webs:did-webs-service%3a7702:EEOqE46OOSl1k1JO3ggQTGuQR3nnWE8bYjOPnJ53m8CP",
    "did:keri:EEOqE46OOSl1k1JO3ggQTGuQR3nnWE8bYjOPnJ53m8CP"
  ]
}
```

:::

### Basic KERI event details

This section is normative.

[DID Documents](#did-documents) introduced the core [[ref: KERI event stream]]
and related DID Document concepts. This section provides additional details
regarding the basic types of KERI events and how they relate to the DID
document.

#### Key state events

1. When processing the KERI event stream `did:webs` MUST account for two
   broad types of key state events (KERI parlance is 'establishment
   events') that can alter the key state of the AID.
1. Any change in key state of the AID MUST be reflected in the DID document.
1. If a key state event does not commit to a future set of rotation key
   hashes, then the AID SHALL NOT be rotated to new keys in the future
   (KERI parlance is that the key state of the AID becomes
   'non-transferrable').
1. If a key state event does commit to a future set of rotation key hashes,
   then any future key state rotation MUST be to those commitment keys.
   This foundation of [[ref: pre-rotation]] is post-quantum safe and allows
   the `did:webs` controller to recover from key compromise.
1. The [[ref: Inception event]] MUST be the first event in the [[ref: KEL]]
   that establishes the AID.
    1. This MUST define the initial key set
    1. If the controller(s) desire future key rotation (transfer) then the
       inception event MUST commit to a set of future rotation key hashes.
    1. When processing the [[ref: KERI event stream]], if there are no
       rotation events after the inception event, then this is the current
       key state of the AID and MUST be reflected in the DID Document as
       specified in [Verification Methods](#verification-methods) and
       [Verification Relationships](#verification-relationships).
1. [[ref: Rotation events]] MUST come after inception events.
1. If the controller(s) desires future key rotation (transfer) then the
   rotation event MUST commit to a set of future rotation key hashes.
1. Rotation events MUST only change the key state to the previously
   committed to rotation keys.
1. Either the inception event or the last rotation event, if any, is the
   current key state of the AID and MUST be reflected in the DID Document
   as specified in [Verification Methods](#verification-methods) and
   [Verification Relationships](#verification-relationships).

::: informative KERI event references
You can learn more about the inception event in the [KERI
specification](#KSWG-KERI) and you can see an example inception event. To learn about
future rotation key commitment, see the sections about
[pre-rotation](#pre-rotation) and the [KERI specification](#KSWG-KERI).

You can learn more about rotation events in the [KERI specification](#KSWG-KERI) and you
can see an example rotation event. To learn about future rotation key
commitment, see the sections about [pre-rotation](#pre-rotation) and the
[KERI specification](#KSWG-KERI).
:::

### Delegation KERI event details

This section focuses on delegation relationships between KERI AIDs.
[DID Documents](#did-documents) introduced the core [[ref: KERI event stream]]
and related DID Document concepts. This section provides additional details
regarding the types of KERI delegation events and how they relate to the DID
document. See [Basic KERI event details](#basic-keri-event-details) for
further detail on basic KERI event types including how they relate to the
DID document.

#### Delegation key state events

1. All delegation relationships MUST start with a delegated inception event.
1. Any change to the delegated inception event key state or
   delegated rotation event key state MUST be the result of a delegated
   rotation event.

::: informative Delegation event summaries
Delegated [[ref: inception event]]: Establishes a delegated identifier.
Either the delegator or the delegate can end the delegation commitment.

Delegated [[ref: rotation event]]: Updates the delegated identifier
commitment. Either the delegator or the delegate can end the delegation
commitment.

See the [KERI specification](#KSWG-KERI) for an example of a delegated inception
and rotation events.
:::

Delegation service endpoints in the DID document are defined in the next
section.

### Service Endpoint Event Details

This section is normative.

In did:webs, KERI-derived service endpoints are defined by **Location Scheme**
(`/loc/scheme`) reply (`rpy`) messages and, for roles other than witness,
**Endpoint Role Authorization** (`/end/role/add`) `rpy` messages in the
[[ref: KERI event stream]]. Location Scheme records declare URL(s) for a
given scheme for an AID; Endpoint Role Authorization relates a role (e.g.
mailbox, agent) of one AID to another. See
[KERI Service Endpoints as DID Document Metadata](#keri-service-endpoints-as-did-document-metadata).

When the event stream (or equivalent key state and endpoint data) for a
`did:webs` DID establishes a witness, mailbox, or agent the DID document
MUST include the associated service endpoint(s) in its `service` array.

#### Witness Service Endpoint

1. A witness service endpoint is produced when (1) the controller AID's
   [[ref: KEL]] designates the witness in its witness list (inception or
   latest rotation event `b` field), and (2) one or more Location Scheme
   `rpy` messages with `r` `/loc/scheme` declare URLs for that witness AID
   (`a.eid`) per scheme. The witness role is thus established by key state,
   not by an Endpoint Role Authorization `rpy`.
2. The DID document service entry SHALL use `type` `witness`, `id` relative
   to the DID of the form `#<witness-aid>/witness`, and `serviceEndpoint`
   as an object whose keys are scheme names and values are the declared
   URLs.

Location Scheme examples (witness AID declares https and tcp URLs):

```json
{
  // ...
  "t": "rpy",
  "r": "/loc/scheme",
  "a": {
    "eid": "BJqHtDoLT_K_XyOgr2ejBOqD9276TYMTg2EEqWKs-V0q",
    "scheme": "https",
    "url": "https://wit1.example.com:5641/"
  }
}
```

```json
{
  // ...
  "t": "rpy",
  "r": "/loc/scheme",
  "a": {
    "eid": "BJqHtDoLT_K_XyOgr2ejBOqD9276TYMTg2EEqWKs-V0q",
    "scheme": "tcp",
    "url": "tcp://wit1.example.com:5631/"
  }
}
```

Resulting witness service entry:

```json
{
  "id": "#BJqHtDoLT_K_XyOgr2ejBOqD9276TYMTg2EEqWKs-V0q/witness",
  "type": "witness",
  "serviceEndpoint": {
    "https": "https://wit1.example.com:5641/",
    "tcp": "tcp://wit1.example.com:5631/"
  }
}
```

#### Mailbox Service Endpoint

1. A mailbox service endpoint is produced when (1) an Endpoint Role
   Authorization `rpy` with `r` `/end/role/add` and `a.role` `mailbox`
   designates the mailbox AID (`a.eid`) for the controller AID (`a.cid`),
   and (2) one or more Location Scheme `rpy` messages with `r`
   `/loc/scheme` declare URLs for that mailbox AID. Implementations obtain
   mailbox endpoints from Endpoint Role data (e.g. KERI `ends` table keyed
   by controller and role) plus Location Scheme data (e.g. `locs` table).
2. The DID document service entry SHALL use `type` `mailbox`, `id` relative
   to the DID of the form `#<mailbox-aid>/mailbox`, and `serviceEndpoint` as
   an object mapping scheme names to URLs (or a single URL when only one
   scheme applies).

Endpoint Role Authorization example (controller designates mailbox):

```json
{
  // ...
  "t": "rpy",
  "r": "/end/role/add",
  "a": {
    "cid": "Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
    "role": "mailbox",
    "eid": "BJqHtDoLT_K_XyOgr2ejBOqD9276TYMTg2EEqWKs-V0q"
  }
}
```

Location Scheme example (mailbox AID declares http URL):

```json
{
  // ...
  "t": "rpy",
  "r": "/loc/scheme",
  "a": {
    "eid": "BJqHtDoLT_K_XyOgr2ejBOqD9276TYMTg2EEqWKs-V0q",
    "scheme": "http",
    "url": "http://mailbox.example.com:5635/"
  }
}
```

Resulting mailbox service entry:

```json
{
  "id": "#BJqHtDoLT_K_XyOgr2ejBOqD9276TYMTg2EEqWKs-V0q/mailbox",
  "type": "mailbox",
  "serviceEndpoint": {
    "https": "https://mailbox.example.com:5635/",
  }
}
```

#### Delegator Service Endpoint

If the first event in the [[ref: KEL]] for a `did:webs` DID is a delegated
inception event of type `dip` then it MUST include a delegator service
endpoint in its DID document as follows.

1. A delegated AID MUST include a service endpoint in its DID document that
   references its delegator.
1. When a delegator service endpoint is present, it MUST conform to the
   following requirements:
    1. The service `type` property MUST be set to `DelegatorOOBI`.
    1. The service `id` property MUST be the [[ref: self-addressing identifier]]
       ([[ref: SAID]]) of the seal (anchor
       block) in the delegator's [[ref: KEL]] that commits to the delegate's
       delegated inception event.
    1. The service `serviceEndpoint` property MUST be a valid
       [[ref: out-of-band introduction]] ([[ref: OOBI]]) URL that resolves to
       the delegator's AID.
1. The delegator service endpoint enables [[ref: verifiers]] to discover and validate
   the delegation relationship by retrieving the delegator's [[ref: KEL]].

For example, a `did:webs` DID that is a delegated AID MUST include, in its
`service` array of the DID document, a delegator service endpoint similar
to the following:

```json
{
  "service": [{
    "id": "EDEvmKvGFjuip-J5dDw7sbVHxXA22s-pBO764CivsFt4",
    "type": "DelegatorOOBI",
    "serviceEndpoint": "http://example.com:3902/oobi/ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe"
  }]
}
```

::: informative Delegator endpoint example explanation
In this example, the `id` field contains the [[ref: SAID]] of the seal in the
delegator's [[ref: KEL]] that anchors the delegation commitment, and the
`serviceEndpoint` provides the [[ref: OOBI]] URL to retrieve the delegator's
key state so that the delegator's KEL may be searched for the delegation
seal referred to by the `id` property.
:::

#### Agent Service Endpoint

1. An agent service endpoint is produced when (1) an Endpoint Role
   Authorization `rpy` with `r` `/end/role/add` and `a.role` `agent`
   designates the agent AID (`a.eid`) for the controller AID (`a.cid`),
   and (2) one or more Location Scheme `rpy` messages with `r`
   `/loc/scheme` declare URLs for that agent AID. Implementations obtain
   agent endpoints from Endpoint Role data (e.g. KERI `ends` table) plus
   Location Scheme data (e.g. `locs` table).
2. The DID document service entry SHALL use `type` `KeriAgent` (or `agent`
   where registered) and `serviceEndpoint` as an object mapping scheme names
   to URLs or a single URL, consistent with
   [KERI Service Endpoints as DID Document Metadata]
   (#keri-service-endpoints-as-did-document-metadata).

Endpoint Role Authorization example (controller designates agent):

```json
{
  // ...
  "t": "rpy",
  "r": "/end/role/add",
  "a": {
    "cid": "Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
    "role": "agent",
    "eid": "BJqHtDoLT_K_XyOgr2ejBOqD9276TYMTg2EEqWKs-V0q"
  }
}
```

Location Scheme example (agent AID declares http URL):

```json
{
  // ...
  "t": "rpy",
  "r": "/loc/scheme",
  "a": {
    "eid": "BJqHtDoLT_K_XyOgr2ejBOqD9276TYMTg2EEqWKs-V0q",
    "scheme": "http",
    "url": "http://agent.example.com:5636/"
  }
}
```

Resulting agent service entry:

```json
{
  "id": "#BJqHtDoLT_K_XyOgr2ejBOqD9276TYMTg2EEqWKs-V0q/agent",
  "type": "agent",
  "serviceEndpoint": {
    "https": "https://agent.example.com:5636/",
  }
}
```

### Designated Aliases

1. An AID controller SHALL specify the [[ref: designated aliases]] that will
   be listed in the `equivalentId` and `alsoKnownAs` properties by issuing a
   Designated aliases verifiable attestation as an ACDC.
    1. This attestation MUST contain a set of [[ref: AID controlled identifiers]]
       that the AID controller authorizes.
    1. If the identifier is a `did:webs` identifier then it is truly
       equivalent and MUST be listed in the `equivalentId` property.
    1. If the identifier is a DID then it MUST be listed in the `alsoKnownAs` property.

#### Designated Aliases event details

::: informative Designated aliases example
This is an example [[ref: designated aliases]] [[ref: ACDC]] attestation
showing five designated aliases:

```json
{
  "v": "ACDC10JSON000574_",
  "d": "EA7VoZA6B9hmhmJAuWUU4lNxRBIQ3sv6JZXGeLnJCRGZ",
  "i": "EEOqE46OOSl1k1JO3ggQTGuQR3nnWE8bYjOPnJ53m8CP",
  "ri": "EBXOxFQrvOxBPb7DiRaTlZNLvX5RbL5cO05MdA35Z0vl",
  "s": "EN6Oh5XSD5_q2Hgu-aqpdfbVepdpYpFlgz6zvJL5b_r5",
  "a": {
    "d": "ELt92UqnBvlk8UOWpK2uUWlXEohe6j_JEVP4zvRY3HB-",
    "dt": "2026-02-06T21:44:36.788115+00:00",
    "ids": [
      "did:web:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
      "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
      "did:web:example.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
      "did:web:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
      "did:webs:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe"
    ]
  },
  "r": {
    ...rules section
  }
}
```

The resulting DID document based on the [[ref: designated aliases]]
attestation above, contains:

* `alsoKnownAs` identifiers as follows:
  * the authorized `did:web` version of the identifier.
  * `did:web:example.com` is an alternative, authorized `did:web` identifier.
  * the `did:webs:foo.com` identifier offers an alternative `did:webs` endpoint.
  * the authorized `did:web` equivalent of the `did:webs:foo.com` identifier.
  * the `did:keri` identifier is always included in `alsoKnownAs` (see [Also Known As](#also-known-as)).
* An `equivalentId` metadata for the `did:webs:foo.com` identifier (since it shares an AID but has differnet host information AND is a `did:webs` identifier. Note that only `did:webs` identifiers should be included in `equivalentId`. See [Use of `equivalentId`](#use-of-equivalentid)).

```json
{
    "didDocument": {
        "id": "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
        "verificationMethod": [
            {
                "id": "#DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr",
                "type": "JsonWebKey",
                "controller": "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
                "publicKeyJwk": {
                    "kid": "DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr",
                    "kty": "OKP",
                    "crv": "Ed25519",
                    "x": "evT4j6Yw3uHpwsw5NEmSR8-4x3S-BA-s6Thjd51oeOs"
                }
            }
        ],
        "service": [],
        "alsoKnownAs": [
            "did:web:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:web:example.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:web:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:webs:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:keri:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe"
        ]
    },
    "didResolutionMetadata": {
        "contentType": "application/did+json",
        "retrieved": "2024-04-01T17:43:24Z"
    },
    "didDocumentMetadata": {
        "witnesses": [],
        "versionId": "2",
        "equivalentId": [
            "did:webs:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe"
        ],
        "didDocUrl": "http://did-webs-service:7676/ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe/did.json",
        "keriCesrUrl": "http://did-webs-service:7676/ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe/keri.cesr"
    }
}
```
:::

## DID Parameters

This section is normative.

This section describes the support of the `did:webs` method for certain
DID parameters.

### Support for `versionId`

The `did:webs` DID method supports the `versionId` DID parameter. This DID
parameter is defined in the [DID specification](https://www.w3.org/TR/did-core/#did-parameters).

This allows clients to instruct a DID Resolver to return a specific version
of a DID document, as opposed to the latest version. The `did:webs` DID
method is ideally suited for this functionality, since a continuous,
self-certifying stream of events lies at the heart of the DID method's
design, see section [KERI Fundamentals](#keri-fundamentals).

1. Valid values for this DID parameter MUST be the sequence numbers of
   events in the [[ref: KERI event stream]].
1. When a `did:webs` DID is resolved with this DID parameter, a
   `did:webs` resolver MUST construct the DID document based on an AID's
   associated KERI events from the KERI event stream only up to
   (and including) the event with the sequence number (i.e. the `s` field)
   that corresponds to the value of the `versionId` DID parameter.

::: informative versionId example
See section [DID Documents](#did-documents) for details.

Example:

```text
did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M?versionId=1
```

:::

### Support for `transformKeys`

The `did:webs` DID method supports the `transformKeys` DID parameter.
This DID parameter is defined as a DID method
[extension](https://github.com/decentralized-identity/did-spec-extensions/blob/main/parameters/transform-keys.md).

1. This parameter MUST be implemented for a DID Resolver to return
   verification methods in a DID document in a desired format, such as
   `JsonWebKey` or `Ed25519VerificationKey2020`.

::: informative transformKeys example
Example:

```text
did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M?transformKeys=CesrKey
```

:::

#### `CesrKey` and `publicKeyCesr`

This specification defines the following extensions to the DID document
data model in accordance with the [DID Spec Registries](#W3C-DID-REGISTRIES):

1. Extension verification method `type` `CesrKey` MAY be available in a
   `did:webs` DID document to express a public key encoded in [[ref: CESR]]
   format.
1. Extension verification method property `publicKeyCesr` MAY be available
   in a `did:webs` DID document to provide a string value whose content is
   the CESR representation of a public key.
1. The verification method type `CesrKey` MAY be used as the value of the
   `transformKeys` DID parameter.

::: informative CesrKey example
For example, a KERI AID with only the following inception event in its KEL:

```json
{
  "v": "KERI10JSON0001b7_",
  "t": "icp",
  "d": "Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
  "i": "Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
  "s": "0",
  "kt": "1",
  "k": [
    "1AAAAg299p5IMvuw71HW_TlbzGq5cVOQ7bRbeDuhheF-DPYk",  // Secp256k1 Key
    "DA-vW9ynSkvOWv5e7idtikLANdS6pGO2IHJy7v0rypvE",      // Ed25519 Key
    "DLWJrsKIHrrn1Q1jy2oEi8Bmv6aEcwuyIqgngVf2nNwu"       // Ed25519 Key
  ],
  // ...
}
```

and given the following the DID URL:

```text
did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M?transformKeys=CesrKey
```

would result in a DID document with the following verification methods array:

```json
{
  "verificationMethod": [
    {
      "id": "#1AAAAg299p5IMvuw71HW_TlbzGq5cVOQ7bRbeDuhheF-DPYk",
      "type": "CesrKey",
      "controller": "did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
      "publicKeyCesr": "1AAAAg299p5IMvuw71HW_TlbzGq5cVOQ7bRbeDuhheF-DPYk"
    },
    {
      "id": "#DA-vW9ynSkvOWv5e7idtikLANdS6pGO2IHJy7v0rypvE",
      "type": "CesrKey",
      "controller": "did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
      "publicKeyCesr": "DA-vW9ynSkvOWv5e7idtikLANdS6pGO2IHJy7v0rypvE"
    },
    {
      "id": "#DLWJrsKIHrrn1Q1jy2oEi8Bmv6aEcwuyIqgngVf2nNwu",
      "type": "CesrKey",
      "controller": "did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M",
      "publicKeyCesr": "DLWJrsKIHrrn1Q1jy2oEi8Bmv6aEcwuyIqgngVf2nNwu"
    }
  ]
}
```

:::

## DID Metadata

This section is normative.

This section describes the support of the `did:webs` method for metadata,
including [[ref: DID resolution metadata]] and [[ref: DID document metadata]]. 
This metadata is returned by a DID Resolver in addition to the
DID document. Also see the [DID Resolution](https://w3c.github.io/did-resolution/)
specification for further details.

### DID Resolution Metadata

At the moment, this specification does not define the use of any specific
[[ref: DID resolution metadata]] properties in the `did:webs` method, but
may in the future include various metadata, such as which KERI Watchers were
used during the resolution process.

### DID Document Metadata

This section of the specification defines how various DID document metadata
properties are used by the `did:webs` method.

#### Use of `versionId`

The `versionId` DID document metadata property indicates the current
version of the DID document that has been resolved.

1. The `did:webs` versionId MUST be the sequence number (i.e. the `s`
   field) of the last event in the [[ref: KERI event stream]] that was used
   to construct the DID document according to the rules in section [DID Documents](#did-documents).
1. If the DID parameter `versionId` (see section [Support for `versionId`](#support-for-versionid))
   was used when resolving the `did:webs` DID, and if the DID Resolution process was successful, then
   this corresponding DID document metadata property MUST be guaranteed to
   be equal to the value of the DID parameter.

Example:

```json
{
  "didDocument": {
    "id": "did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M"
    // ... other properties
  },
  "didResolutionMetadata": {
  },
  "didDocumentMetadata": {
    "versionId": "2"
  }
}
```

#### Use of `nextVersionId`

The `nextVersionId` DID document metadata property indicates the next
version of the DID document after the version that has been resolved.

1. The `did:webs` `nextVersionId` MUST be the sequence number (i.e. the `s`
   field) of the next event in the [[ref: KERI event stream]] after the
   last one that was used to construct the DID document according to the
   rules in section [DID Documents](#did-documents).
1. This DID document metadata property MUST be present if the DID
   parameter `versionId` (see section [Support for `versionId`](#support-for-versionid))
   was used when resolving the `did:webs` DID, and if the value of that DID parameter was not the
   sequence number of the last event in the KERI event stream.

Example:

```json
{
  "didDocument": {
    "id": "did:webs:example.com:Ew-o5dU5WjDrxDBK4b4HrF82_rYb6MX6xsegjq4n0Y7M"
    // ... other properties
  },
  "didResolutionMetadata": {
  },
  "didDocumentMetadata": {
    "versionId": "1",
    "nextVersionId": "2"
  }
}
```

#### Use of `equivalentId`

The `equivalentId` DID document metadata property indicates other DIDs that
refer to the same subject and are logically equivalent to the DID that has
been resolved. It is similar to the `alsoKnownAs` DID document property (see
section [Also Known As](#also-known-as)), but it has even stronger
semantics, insofar as the logical equivalence is guaranteed by the DID
method itself.

1. The `did:webs` `equivalentId` metadata property SHOULD contain a list of
   the controller AID [[ref: designated aliases]] `did:webs` DIDs that differ
   in the [[ref: host]] and/or port portion of the [[ref: method-specific identifier]]
   but share the same AID. Also see section [[ref: AID controlled identifiers]].
1. `equivalentId` depends on the controller AIDs array of [[ref: designated aliases]]. 
   A `did:webs` identifier MUST not verify unless it is found in
   the `equivalentId` metadata that corresponds to the Designated aliases.

> Note that [[ref: AID controlled identifiers]] like `did:web` and
> `did:keri` identifiers with the same AID are not listed in `equivalentId`
> because they do not have the same DID method. A `did:web` identifier with
> the same domain and AID does not have the same security characteristics as
> the `did:webs` identifier. Conversely, a `did:keri` identifier with the
> same AID has the same security characteristics but not the same dependence
> on the web. For these reasons, they are not listed in `equivalentId`.

Example:

```json
{
    "didDocument": {
        "id": "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
        "verificationMethod": [
            {
                "id": "#DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr",
                "type": "JsonWebKey",
                "controller": "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
                "publicKeyJwk": {
                    "kid": "DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr",
                    "kty": "OKP",
                    "crv": "Ed25519",
                    "x": "evT4j6Yw3uHpwsw5NEmSR8-4x3S-BA-s6Thjd51oeOs"
                }
            }
        ],
        "service": [],
        "alsoKnownAs": [
            "did:web:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:web:example.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:web:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:webs:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe"
        ]
    },
    "didResolutionMetadata": {
        "contentType": "application/did+json",
        "retrieved": "2024-04-01T17:43:24Z"
    },
    "didDocumentMetadata": {
        "witnesses": [],
        "versionId": "2",
        "equivalentId": [
            "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:webs:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe"
        ],
        "didDocUrl": "http://did-webs-service:7676/ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe/did.json",
        "keriCesrUrl": "http://did-webs-service:7676/ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe/keri.cesr"
    }
}
```

## Security Considerations

This section is normative.

There are many security considerations related to web requests, storing
information securely, etc. It is useful to address these considerations
along with the common security threats found on the web.

### Common security threats

1. All `did:webs` features MUST reduce the attack surface against common threats:
    1. Broken Object Level Authorization (BOLA) attacks MUST be eliminated or reduced.
    1. Denial of service (DoS) attacks MUST be eliminated or reduced.
    1. Deletion attacks MUST be eliminated or reduced.
    1. Duplicity detection MUST be available and reliable.
    1. Eclipse attacks MUST be eliminated or reduced.
    1. Forgery attacks MUST be eliminated or reduced.
    1. Impersonation attacks MUST be eliminated or reduced.
    1. Key Compromise attacks MUST be eliminated or reduced.
    1. Malleability attacks MUST be eliminated or reduced.
    1. Replay attacks MUST be eliminated or reduced.

### Using HTTPS

Perfect protection from eavesdropping is not possible with HTTPS, for
various reasons.

1. URLs of DID documents and [[ref: KERI event streams]] SHOULD be hosted
   in a way that embodies accepted cybersecurity best practice. This is not
   strictly necessary to guarantee the authenticity of the data. However,
   the usage:
    1. MUST safeguard privacy
    1. MUST discourage denial of service
    1. MUST work in concert with defense-in-depth mindset
    1. MUST aid regulatory compliance
    1. MUST allow for high-confidence fetches of the DID document and a KERI
       event stream
1. A [[ref: host]] that uses a fully qualified domain name of the
   [[ref: method-specific identifier]] MUST be secured by a TLS/SSL
   certificate.
    1. The fully qualified domain name MUST match the common name used in the
       SSL/TLS certificate.
    1. The common name in the SSL/TLS certificate from the server MUST
       correspond to the way the server is referenced in the URL. This means
       that if the URL includes `www.example.com`, the common name in the
       SSL/TLS certificate must be `www.example.com` as well.
1. Unlike `did:web`, the URL MAY use an IP address instead.
    1. If it does, then the common name in the certificate MUST be the IP
       address as well.
1. Essentially, the URL and the certificate MUST NOT identify the server in
   contradictory ways; subject to that constraint, how the server is
   identified is flexible.
    1. The server certificate MAY be self-issued
    1. OR it MAY chain back to an unknown certificate authority. However, to
       ensure reasonable security hygiene, it MUST be valid. This has two
       meanings, both of which are required:
1. The certificate MUST satisfy whatever requirements are active in the
   client, such that the client does accept the certificate and use it to
   build and communicate over the encrypted HTTPS session where a DID
   document and KERI event stream are fetched.
1. The certificate MUST pass some common-sense validity tests, even if the
   client is very permissive:
    1. It MUST have a valid signature
    1. It MUST NOT be expired or revoked or deny-listed
    1. It MUST NOT have any broken links in its chain of trust.
1. If a URL of a DID document or KERI event streams results in a redirect,
   each URL MUST satisfy the same security requirements.

### International Domain Names

1. As with `did:web`, implementers of `did:webs` SHOULD consider how non-ASCII
   characters manifest in URLs and DIDs.
    1. `did:webs` MUST follow the [DID Core](#W3C-DID-CORE) identifier syntax which
       does not allow the direct representation of such characters in method
       name or method specific identifiers. This prevents a `did:webs` value
       from embodying a homograph attack.
    1. However, `did:webs` MAY hold data encoded with punycode or percent
       encoding. This means that IRIs constructed from DID values could
       contain non-ASCII characters that were not obvious in the DID,
       surprising a casual human reader.
    1. Caution is RECOMMENDED when treating a `did:webs` as the equivalent
       of an IRI.
    1. Treating it as the equivalent of a URL, instead, is RECOMMENDED as it
       preserves the punycode and percent encoding and is therefore safe.

### Concepts for securing `did:webs` information

The following security concepts are used to secure the data, files,
signatures and other information in `did:webs`.

1. All security features and concepts in `did:webs` MUST use one or more of
   the following mechanisms:
    1. All data that requires the highest security MUST be [[ref: KEL]]
       backed. This includes any information that needs to be end-verifiably
       authentic over time:
        1. All [[ref: ACDCs]] used by a `did:webs` identifier MUST be one
           of the following:
            1. MAY be anchored to a KEL directly.
            1. MAY be anchored indirectly through a [[ref: TEL]] that itself
               is anchored to a KEL.
    1. All data that does not need to incur the cost of [[ref: KEL]] backing
       for secuirty but can benefit from the latest data-state such as a
       distributed data-base MUST use _Best Available Data - Read, Update,
       Nullify_ ([[ref: BADA-RUN]]).
        1. BADA-RUN information MUST be ordered in a consistent way, using the following:
            1. date-time MUST be used.
            1. key state MUST be used.
        1. Discovery information MAY use BADA-RUN because the worst-case
           attack on discovery information is a DDoS attack where nothing
           gets discovered.
            1. The controller(s) of the AID for a `did:webs` identifier MAY use
               BADA-RUN for service end-points as discovery mechanisms.
    1. All data that does not need the security of being KEL backed nor
       BADA-RUN SHOULD be served using
       _[[ref: KERI Request Authentication Mechanism]]_ ([[ref: KRAM]]).
        1. For a `did:webs` resolver to be trusted it SHOULD use KRAM to
           access the service endpoints providing KERI event streams for
           verification of the DID document.

#### Reducing the attack surface

::: informative Reducing the attack surface

The above considerations have lead us to focus on KEL backed DID document
blocks and data (designated alias ACDCs, signatures, etc) so that the trusted
(local) did:webs resolver is secure. Any future features that could
leverage BADA-RUN and [[ref: KRAM]] should be considered carefully according
to the above considerations.

See the implementors guide for more details about KEL backed, BADA-RUN, and KRAM:

* [On-Disk Storage](#on-disk-storage)
* [Alignment of Information to Security Posture](#alignment-of-information-to-security-posture)
* [Applying the concepts of KEL, BADA-RUN, and KRAM](#applying-the-concepts-of-kel)

:::

## Privacy Considerations

::: informative Privacy Considerations

This section addresses the privacy considerations from
[RFC6973](https://datatracker.ietf.org/doc/html/rfc6973) section 5.
For privacy considerations related to web infrastructure, see
[`did:web` privacy considerations](https://w3c-ccg.github.io/did-method-web/#security-and-privacy-considerations).
Below we discuss privacy considerations related the KERI infrastructure.

### Surveillance

In KERI, a robust [[ref: witness]] network along with consistent [[ref: witness]]
rotation
provides protection from monitoring and association of an individual's
activity inside a KERI network.

### Stored Data Compromise

For resolvers that simply discover the Key State endorsed by another party
in a discovery network, caching policies of that network would guide stored
data security considerations.  In the event that a resolver is also the
endorsing party, meaning they have their own KERI identifier and are
verifying the KEL and signing the Key State themselves, leveraging the
facilities provided by the KERI protocol (key rotation, witness
maintenance, multi-sig) should be used to protect the identities used to
sign the Key State.

### Unsolicited Traffic

DID Documents are not required to provide endpoints and thus not subject to
unsolicited traffic.

### Misattribution

This DID Method relies on KERI's duplicity detection to determine when the
non-repudiable controller of a DID has been inconsistent and can no longer
be trusted.  This establishment of non-repudiation enables consistent
attribution.

### Correlation

The root of trust for KERI identifiers is entropy and therefore offers no
direct means of correlation.  In addition, KERI provides two modes of
communication, [[ref: direct mode]] and [[ref: indirect mode]]. [[ref: direct mode]]
allows for
pairwise (n-wise as well) relationships that can be used to establish
private relationships.

See the KERI specification for
[more information about direct and indirect modes](https://trustoverip.github.io/kswg-keri-specification/#introduction).

### Identification

The root of trust for KERI identifiers is entropy and therefore offers no
direct means of identification.  In addition, KERI provides two modes of
communication, [[ref: direct mode]] and [[ref: indirect mode]]. [[ref: direct mode]]
allows for
pairwise (n-wise as well) relationships that can be used to establish
private relationships.

See the KERI specification for
[more information about secure bindings and prefix derivation](https://trustoverip.github.io/kswg-keri-specification/#keris-secure-bindings)

### Secondary Use

The Key State made available in the metadata of this DID method is generally
available and can be used by any party to retrieve and verify the state of
the [[ref: key event receipt log]] ([[ref: KERL]]) for the given
[[ref: identifier]].

### Disclosure

No data beyond the Key State for the identifier is provided by this DID
method.

### Exclusion

This DID method provides no opportunity for [correlation](#correlation),
[identification](#identification) or [disclosure](#disclosure) and therefore
there is no opportunity to exclude the controller from knowing about data
that others have about them.

::: 

## IANA Considerations

This section is normative.

[[ref: KERI event streams]] use the registered
[application/cesr media type with IANA][iana-cesr]. The media type is
`application/cesr`. The registration follows the template in
[RFC6838](https://tools.ietf.org/html/rfc6838).

[iana-cesr]:
https://www.iana.org/assignments/provisional-standard-media-types/provisional-standard-media-types.xml


## Implementors Guide

These sections are informative.

### Key Agreement

::: informative Key Agreement
Key Agreement

There are multiple ways to establish key agreement in KERI. We detail common
considerations and techniques:

* If the 'k' field references a Ed25519 key, then key agreement may be
   established using the corresponding x25519 key for Diffie-Helman key
   exchange.
* If the key is an ECDSA or other NIST algorithms key then it will be the
   same key for signatures and encryption and can be used for key agreement.
* _BADA-RUN for key agreement:_ Normally in KERI we would use
   [[ref: BADA-RUN]], similar to how we specify endpoints, [[ref: host]]
   migration info, etc. This would allow the controller to specify any Key
   Agreement key, without unnecessarily adding KERI events to their
   [[ref: KEL]].
* _Key agreement from `k` field keys:_ It is important to note that KERI is
   cryptographically agile and can support a variety of keys and signatures.
* _Key agreement anchored in KEL:_ It is always possible to anchor arbitrary
   data, like a key agreement key, to the KEL.
  * The best mechanism is to anchor an [[ref: ACDC]] to a [[ref: TEL]] which
    is anchored to the KEL. The data would be the basis of the key agreement
    information. The concept is similar to how we anchor
    [[ref: designated aliases]] as
    [verifiable data on a TEL](#verifiable-data-on-a-tel).
:::

### Other Key Commitments

::: informative Other Key Commitments
Key Commitments

This section is informative: Data structures similar to Location Scheme and
Endpoint Authorizations and managed in KERI using [[ref: BADA-RUN]] may be
created and used for declaring other types of keys, for example encryption
keys, etc

To support new data structures, propose them in KERI and detail the
transformation in the spec.
:::

### On-Disk Storage

::: informative On-Disk Storage
On-Disk Storage

This section is informative: Both [[ref: KEL backed data]] and [[ref: BADA-RUN]]
security approaches are suitable for storing information on disk because both
provide a link between the keystate and date-time on some data when a
signature by the source of the data was created. [[ref: BADA-RUN]] is too
weak for important information because an attacker who has access to the
database on disk can overwrite data on disk without being detected by a
verifier hosting the on-disk data either through a replay of stale data (data
regression attack) or if in addition to disk access the attacker has
compromised a given key state, then the attacker can forge new data with a
new date-time stamp for a given compromised key and do a regression attack
so that the last seen key state is the compromised key state.

With BADA, protection from a deletion (regression) attack requires redundant
disk storage. At any point in time where there is a suspicion of loss of
custody of the disk, a comparison to the redundant disks is made and if any
disk has a later event given BADA-RUN rules then recovery from the deletion
attack is possible.

[[ref: KRAM]] on a query is not usable for on disk storage by itself because
it's just a bare signature (the datetime is not of the querier but of the
host at the time of a query). However, the reply to a query can be stored on
disk if the querier applies BADA to the reply. To elaborate, Data obtained
via a KRAMed query-reply may be protected on-disk by being using BADA on the
reply. This is how KERI stores service endpoints. However, KERI currently
only uses BADA for discovery data not more important data. More important
data should be wrapped (containerized) in an [[ref: ACDC]] that is
[[ref: KEL]] backed and then stored on-disk

In the hierarchy of attack surfaces, exposure as on disk (unencrypted) is
the weakest. Much stronger is exposure that is only in-memory. To attack
in-memory usually means compromising the code supply chain which is harder
than merely gaining disk access. Encrypting data on disk does not necessarily
solve attacks that require a key compromise (because decryption keys can be
compromised), and it does not prevent a deletion attack. Encryption does
not provide authentication protection. However, encryption does protect the
confidentiality of data.

The use of DH key exchange as a weak form of authentication is no more
secure than an HMAC for authentication. It is sharing secrets, so anyone
with the secret can impersonate any other member of the group that has the
shared secret.

Often, DID methods have focused on features that erode security
characteristics. The paper
[Five DID Attacks](https://github.com/WebOfTrustInfo/rwot11-the-hague/blob/master/final-documents/taking-out-the-crud-five-fabulous-did-attacks.pdf)
highlights some attacks to which `did:webs` should NOT be vulnerable. So
when a pull request exposes `did:webs` to a known attack, it should not be
accepted.
:::

### Alignment of Information to Security Posture

::: informative Alignment of Information to Security Posture
Alignment of Information to Security Posture

This section is informative: As a general security principle each block of
information should have the same security posture for all the sub-blocks.
One should not attempt to secure a block of information that mixes security
postures across its constituent sub-blocks. The reason is that the security
of the block can be no stronger than the weakest security posture of any
sub-block in the block. Mixing security postures forces all to have the
lowest common denominator security. The only exception to this rule is if
the block of information is purely informational for discovery purposes and
where it is expected that each constituent sub-block is meant to be
verified independently.

This means that any recipient of such a block of information with mixed
security postures across its constituent sub-blocks must explode the block
into sub-blocks and then independently verify the security of each
sub-block. However, this is only possible if the authentication factors for
each sub-block are provided independently. Usually when information is
provided in a block of sub-blocks, only one set of authentication factors
are provided for the block as a whole and therefore there is no way to
independently verify each sub-block of information.

Unfortunately, what happens in practice is that users are led into a false
sense of security because they assume that they don't have to explode and
re-verify, but merely may accept the lowest common denominator verification
on the whole block of information. This creates a pernicious problem for
downstream use of the data. A downstream use of a constituent sub-block
doesn't know that it was not independently verified to its higher level of
security. This widens the attack surface to any point of down-stream usage.
This is a root cause of the most prevalent type of attack called a BOLA.
:::

### Applying the concepts of KEL, BADA-RUN, and KRAM to `did:webs`

::: informative Applying the concepts of KEL
Applying the concepts of KEL, BADA-RUN, and KRAM to `did:webs`

This section is informative. Lets explore the implications of applying these
concepts to various `did:webs` elements.
Using [[ref: KEL]] backed elements in a DID doc simplifies the security
concerns. However, future discovery features related to endpoints might
consider BADA-RUN. For instance, 'whois' data could be used with
[[ref: BADA-RUN]] whereas did:web aliases should not because it could lead
to an impersonation attack. We could have a DID document that uses
BADA-RUN if we modify the DID CRUD semantics to be RUN semantics without
necessarily changing the verbs but by changing the semantics of the verbs.
Then any data that fits the security policy of BADA (i.e. where BADA is
secure enough) can be stored in a DID document as a database in the sky. For
sure this includes service endpoints for discovery. One can sign with
[[ref: CESR]] or JWS signatures. The payloads are essentially KERI reply
messages in terms of the fields (with modifications as needed to be more
palatable), but are semantically the same. The DID doc just relays those
replies. Anyone reading from the DID document is essentially getting a KERI
reply message, and they then should apply the BADA rules to their local
copy of the reply message.

To elaborate, these security concepts point us to modify the DID CRUD
semantics to replicate RUN semantics. _Create_ becomes synonymous with
_Update_ where Update uses the RUN update. _Delete_ is modified to use the
Nullify semantics. _Read_ data is modified so that any recipient of the Read
response can apply BADA to its data (Read is a GET). So we map the CRUD of
DID docs to RUN for the `did:webs` method. Now you have reasonable security
for things like signed data. If its [[ref: KEL]] backed data you could even
use an [[ref: ACDC]] as a data attestation for that data and the did
resolver would become a caching store for [[ref: ACDCs]] issued by the AID
controller.

Architecturally, a Read (GET) from the did resolver acts like how KERI reply
messages are handled for resolving service endpoint discovery from an
[[ref: OOBI]]. The query is the read in RUN and so uses KRAM. The reply is
the response to the READ request. The controller of the AID updates the DID
resolvers cache with updates(unsolicited reply messages). A trustworthy DID
resolver applies the BADA rules to any updates it receives. Optionally the
DID resolver may apply [[ref: KRAM]] rules to any READ requests to protect
it from replay attacks.

In addition, a DID doc can be a discovery mechanism for an ACDC caching
server by including an index (label: said) of the [[ref: SAIDs]] of the
[[ref: ACDCs]] held in the resolvers cache.
:::

### The set of KERI features needed

::: informative The set of KERI features needed
Generally the full featureset of KERI, ACDC, and CESR are needed for `did:webs` 
though specific use cases may depend on a smaller subset of KERI and ACDC.

[[ref: KERI]] means [[ref: key event receipt infrastructure]].
[[ref: ACDC]] means [[ref: authentic chained data container]].
[[ref: CESR]] means [[ref: compact event streaming representation]].

Full support of `did:webs` depends on the following KERI, ACDC, and CESR features:

<div style="overflow-x: auto; white-space: nowrap;">

| Source | Feature                             | Usage in `did:webs`                                                                                                   | Purpose                                                                                                                                      |
|--------|-------------------------------------|-----------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| KERI   | Key Event Log (KEL)                 | `did:webs` DIDs use a KERI AID backed by a KEL.                                                                       | The KEL provides the security for a `did:webs` DID, including key rotation.                                                                  |
| KERI   | Location Scheme Message             | URL Discovery. Part of the `keri.cesr` stream. Service endpoints (witnesses, etc.) use these to list authorized URLs. | Witness service endpoint section construction in the `service` DID doc section providing the backing witness URLs for the `did:webs` AID.    |
| KERI   | Endpoint Role Authorization Message | Service endpoint authorizations. Part of the `keri.cesr` stream. Authorizes an infrastructure component in a role.    | Provides the `type` attribute of service endpoints in the `service` DID doc section for witnesses or other infrastructure components.        | 
| KERI   | Out of Band Identifier (OOBI)       | The `serviceEndpoint` for a delegated identifier uses a KERI OOBI                                                     | Indicates where a `did:webs` resolver would check to retrieve the KERI event stream for a given `did:webs` AID or its delegator.             | 
| ACDC   | Credential Graph (DAG)              | Every `did:webs` DID defines authorized host and path with a designated aliases ACDC (credential)                     | Designated aliases ACDC provides `did:webs` DID host and path segment verifiability.                                                         |  
| KERI   | Key Rotation                        | `versionId` allows requesting a `did:webs` DID document at a point in time as of a given KEL sequence number.         | Long term auditability and verifiability depend on versioned DID documents.                                                                  |
| KERI   | Multi-signature Identifiers         | A `did:webs` DID may use multi-signature identifiers with ConditionalProof2022 for advanced identifier control.       | Enterprise and personal identifier usage often include multi-signature, multi-person setups. Multi-sig facilitates this.                     |
| KERI   | Delegated Identifiers               | The `DelegatorOOBI` service type in `did:webs` relies upon the delegated identifier feature of KERI.                  | Delegated identifiers support structured, hierarchical identifier control often modelled in enterprise, personal, or guardianship scenarios. |
| ACDC   | Transaction Event Log (TEL)         | Anchoring of designated alias credentials to a KEL consists of anchoring TEL events to a KEL for that ACDC.           | TELs are a fundamental, critical component of anchoring an ACDC and are required to be in the`did:webs` DID's `keri.cesr` stream.            |
| CESR   | Text or binary representation       | All `did:webs` DIDs present a CESR-encoded `keri.cesr` stream. Also, an AID is a CESR encoded string.                 | CESR is the text and binary wire protocol fundamental to KERI, ACDC, and `did:webs`                                                          | 

</div>

These basics are
summarized in the [KERI Fundamentals](#keri-fundamentals) section of this
specification. This specification assumes a working knowledge of the
concepts there. The inclusion of KERI in `did:webs` enables a number of
capabilities for securing a `did:webs` identifier, including multi-signature
support, delegated identifier support for signing scalability, and the
creation of [[ref: pre-rotated]] keys to prevent loss of control of the
identifier if the current private key were to be compromised.
You may find the
[did:webs feature dependency diagram](https://github.com/trustoverip/kswg-did-method-webs-specification/blob/main/WEBS_FEATURE_DEPS.md)
helpful as a visual guide to the features and dependencies of `did:webs`.
:::

### Stable identifiers on an unstable web

::: informative Stable identifiers on an unstable web
Stable identifiers on an unstable web

This section is informative: The web is not a very stable place, and
documents are moved around and copied frequently. When two or more companies
merge, often the web presence of some of the merged entities "disappears".
It may not be possible to retain a permanent `did:webs` web location.
The purpose of the history of [[ref: designated aliases]] for the AID is so
that if the `did:webs` DID has been put in long-lasting documents, and its
URL instantiation is redirected or disappears, the controller can explicitly
indicate that the new DID is an `equivalentId` to the old one.

Since the AID is globally unique and references the same identifier,
regardless of the rest of the string that is the full `did:webs`, web
searching could yield either the current location of the DID document, or a
copy of the DID that may be useful. For example, even the
[Internet Archive: Wayback Machine](https://archive.org/web) could be used
to find a copy of the DID document and the [[ref: KERI event stream]] at
some point in the past that may be sufficient for the purposes of the
entity trying to resolve the DID. This specification does not rely on the
Wayback Machine, but it might be a useful `did:webs` discovery tool.

The DID document, [[ref: KERI event stream]] and other files related to a
DID may be copied to other web locations. For example, someone might want
to keep a cache of DIDs they use, or an entity might want to run a
registry of "useful" DIDs for a cooperating group. While the combination of
DID document and KERI event stream make the DID and DID document verifiable,
just as when published in their "intended" location, the absence of the
`did:webs` in the [[ref: designated aliases]] for those locations in the
DID document `equivalentId` means that the controller of the DID is not
self-asserting any sort of tie between the DID and the location to which
the DID-related documents have been copied. In contrast, if the controller
confirms the link between the source and the copy with an `equivalentId`,
the related copies will have to be kept in sync.

Were the AID of a `did:webs` identifier to change, it would be an altogether
new DID, unconnected to the first DID. Furthermore, changing the host or path
of a `did:webs` DID also makes a new DID, albeit with the same backing AID.

A `did:webs` could be moved to use another DID method that uses the AID for
uniqueness and the [[ref: KERI event stream]] for validity, but that is
beyond the scope of this specification.
:::

### KERI event stream chain of custody

::: informative KERI event stream chain of custody
KERI event stream chain of custody

The KERI event stream represents a cryptographic chain of trust originating
from the [[ref: AID]] of the controller to the current operational set of
keys (signing and otherwise) as well as the cryptographic commitments to the
keys the controller will rotate to in the future. The KERI event stream also
contains events that do not alter the [[ref: AID]] key state, but are useful
metadata for the DID document such as, the supported [[ref: hosts]], the
current set of service endpoints, etc. A did:webs resolver produces the
DID document by processing the [[ref: KERI event stream]] to determine the
current key state. We detail the different events in "Basic KERI event
details" and show how they change the DID Document. The mapping from the
KERI event stream to the DID Document properties compose the core of the
did:webs resolver logic.  Understanding the optimal way to update and
maintain the KERI event stream (publish static keri.cesr files, dynamically
generate the keri.cesr resource, etc) is beyond the scope of the spec,
but the [did:webs Reference Implementation](#DIDWEBS-REF-IMPL) of the resolver
demonstrate some of these techniques. The important concept is that the
entire KERI event stream is used to produce and verify the DID document.
:::

### Verifiable data on a TEL

::: informative verifiable data on a TEL
This section is informative: Below is an example highlighting how verifiable
data is anchored to a [[ref: KEL]] using a [[ref: TEL]]. We use this spec's
[[ref: designated aliases]] feature as a real-world example. You can walk
through this example in the [did:webs Reference Implementation](#DIDWEBS-REF-IMPL).
The attestation (self-issued credential) is as follows:

```json
{
    "v": "ACDC10JSON0005f2_",
    "d": "EIGWggWL2IHiUzj1P2YuPA0-Uh55LTIu14KTvVQGrfvT",
    "i": "ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
    "ri": "EAtQJEQMkkvlWxyfLbcLyv4kNeAI5Qsqe65vKIWnHKpx",
    "s": "EN6Oh5XSD5_q2Hgu-aqpdfbVepdpYpFlgz6zvJL5b_r5",
    "a": {
        "d": "EJJjtYa6D4LWe_fqtm1p78wz-8jNAzNX6aPDkrQcz27Q",
        "dt": "2023-11-13T17:41:37.710691+00:00",
        "ids": [
            "did:web:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:web:example.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:web:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:webs:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe"
        ]
    },
    "r": {
        "d": "EEVTx0jLLZDQq8a5bXrXgVP0JDP7j8iDym9Avfo8luLw",
        "aliasDesignation": {
            "l": "The issuer of this ACDC designates the identifiers in the ids field as the only allowed namespaced aliases of the issuer's AID."
        },
        "usageDisclaimer": {
            "l": "This attestation only asserts designated aliases of the controller of the AID, that the AID controlled namespaced alias has been designated by the controller. It does not assert that the controller of this AID has control over the infrastructure or anything else related to the namespace other than the included AID."
        },
        "issuanceDisclaimer": {
            "l": "All information in a valid and non-revoked alias designation assertion is accurate as of the date specified."
        },
        "termsOfUse": {
            "l": "Designated aliases of the AID must only be used in a manner consistent with the expressed intent of the AID controller."
        }
    }
}
```

Now we show that the information is anchored to the KEL in a way that
allows for changes in key state while not invalidating it.  We will:

* chain an interaction event on the KEL, to a registry we call a TEL
* The TEL maintains the 'state' of issued or revoked for the attestation. If
   the controller wants to update the list they would revoke the attestation
   and issue a new attestation with the updated list of aliases that they
   want to designate.
* The TEL must chain to the attestation itself.
This is what forms the end-to-end verifiability from the attestation to the
AID itself. Here is the [[ref: KERI Event Stream]] of each part:
:::

#### The interaction event on the KEL

::: informative interaction event
This interaction event connects the TEL registry to the KEL. Notice the `a`
field with the nested `i` field that references the registry

```json
    "v": "KERI10JSON00013a_",
    "t": "ixn",
    "d": "ED-4iQIVxwMcrTOW6fVs9oPpLTIxtqh_vcvLmE999zsU",
    "i": "ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
    "s": "1",
    "p": "ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
    "a": [
        {
            "i": "EAtQJEQMkkvlWxyfLbcLyv4kNeAI5Qsqe65vKIWnHKpx",
            "s": "0",
            "d": "EAtQJEQMkkvlWxyfLbcLyv4kNeAI5Qsqe65vKIWnHKpx"
        }
    ]
```

:::

#### The TEL registry

::: informative TEL registry
This TEL registry connects the KEL interaction event to the registry status.
Notice the `d` field matches the nested `i` field of the `a` field of the
interaction event demonstrating that they are cryptographically chained
together.

```json
    "v": "KERI10JSON000113_",
    "t": "vcp",
    "d": "EAtQJEQMkkvlWxyfLbcLyv4kNeAI5Qsqe65vKIWnHKpx",
    "i": "EAtQJEQMkkvlWxyfLbcLyv4kNeAI5Qsqe65vKIWnHKpx",
    "ii": "ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
    "s": "0",
    "c": [
        "NB"
    ],
    "bt": "0",
    "b": [],
    "n": "AAfqHwMDdBoIWk_4Z6hvVJuhtvjA_gk8Y9bEUoP_rC_p"
  ```

:::  

#### The attestation status

::: informative issuance TEL event
The current status of the attestation is in the `t` field. `iss` means
issued. Notice the `ri` field cryptographically binds this issued status to
the registry above. Likewise, the `i` field cryptographically binds to the
attestation `d` field.

```json
    "v": "KERI10JSON0000ed_",
    "t": "iss",
    "d": "EJQvCZQYn8oO1z3_f8qhxXjk7TcLol4G3RdHVTwfGV3L",
    "i": "EIGWggWL2IHiUzj1P2YuPA0-Uh55LTIu14KTvVQGrfvT",
    "s": "0",
    "ri": "EAtQJEQMkkvlWxyfLbcLyv4kNeAI5Qsqe65vKIWnHKpx",
    "dt": "2023-11-13T17:41:37.710691+00:00"
```

:::

#### The full KERI Event Stream

::: informative
This snippet demonstrates how these events occur in the full keri.cesr file.
Notice the CESR encoded verifiable signatures that are interleaved between
events.

```json
{
    "v": "KERI10JSON00012b_",
    "t": "icp",
    "d": "ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
    "i": "ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
    "s": "0",
    "kt": "1",
    "k": [
        "DHr0-I-mMN7h6cLMOTRJkkfPuMd0vgQPrOk4Y3edaHjr"
    ],
    "nt": "1",
    "n": [
        "ELa775aLyane1vdiJEuexP8zrueiIoG995pZPGJiBzGX"
    ],
    "bt": "0",
    "b": [],
    "c": [],
    "a": []
}-VAn-AABAADjfOjbPu9OWce59OQIc-y3Su4kvfC2BAd_e_NLHbXcOK8-3s6do5vBfrxQ1kDyvFGCPMcSl620dLMZ4QDYlvME-EAB0AAAAAAAAAAAAAAAAAAAAAAA1AAG2024-04-01T17c40c48d329209p00c00{
    "v": "KERI10JSON00013a_",
    "t": "ixn",
    "d": "ED-4iQIVxwMcrTOW6fVs9oPpLTIxtqh_vcvLmE999zsU",
    "i": "ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
    "s": "1",
    "p": "ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
    "a": [
        {
            "i": "EAtQJEQMkkvlWxyfLbcLyv4kNeAI5Qsqe65vKIWnHKpx",
            "s": "0",
            "d": "EAtQJEQMkkvlWxyfLbcLyv4kNeAI5Qsqe65vKIWnHKpx"
        }
    ]
}-VAn-AABAACNra5mDg7YHFtBeXiwIGqnHkyq7F55FGNYG1wH95akjSWCb1HzNI3E05ufT0HffClDxnJF_DmAUW2SBb0EJeoO-EAB0AAAAAAAAAAAAAAAAAAAAAAB1AAG2024-04-01T17c42c37d704426p00c00{
    "v": "KERI10JSON00013a_",
    "t": "ixn",
    "d": "EBjw0a_L8M0F4xYND99dvahlrkpxODi9Wc9VzUvkhD0t",
    "i": "ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
    "s": "2",
    "p": "ED-4iQIVxwMcrTOW6fVs9oPpLTIxtqh_vcvLmE999zsU",
    "a": [
        {
            "i": "EIGWggWL2IHiUzj1P2YuPA0-Uh55LTIu14KTvVQGrfvT",
            "s": "0",
            "d": "EJQvCZQYn8oO1z3_f8qhxXjk7TcLol4G3RdHVTwfGV3L"
        }
    ]
}-VAn-AABAABo_okwAmWIYWI93EtUONZiEvsGuSRkKnj0mopX_RoXwWHZ_1V5hQ0BxcntsmAi21DbusyCmK-fHwTNtSxUSsoN-EAB0AAAAAAAAAAAAAAAAAAAAAAC1AAG2024-04-01T17c42c39d995867p00c00{
    "v": "KERI10JSON000113_",
    "t": "vcp",
    "d": "EAtQJEQMkkvlWxyfLbcLyv4kNeAI5Qsqe65vKIWnHKpx",
    "i": "EAtQJEQMkkvlWxyfLbcLyv4kNeAI5Qsqe65vKIWnHKpx",
    "ii": "ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
    "s": "0",
    "c": [
        "NB"
    ],
    "bt": "0",
    "b": [],
    "n": "AAfqHwMDdBoIWk_4Z6hvVJuhtvjA_gk8Y9bEUoP_rC_p"
}-VAS-GAB0AAAAAAAAAAAAAAAAAAAAAABED-4iQIVxwMcrTOW6fVs9oPpLTIxtqh_vcvLmE999zsU{
    "v": "KERI10JSON0000ed_",
    "t": "iss",
    "d": "EJQvCZQYn8oO1z3_f8qhxXjk7TcLol4G3RdHVTwfGV3L",
    "i": "EIGWggWL2IHiUzj1P2YuPA0-Uh55LTIu14KTvVQGrfvT",
    "s": "0",
    "ri": "EAtQJEQMkkvlWxyfLbcLyv4kNeAI5Qsqe65vKIWnHKpx",
    "dt": "2023-11-13T17:41:37.710691+00:00"
}-VAS-GAB0AAAAAAAAAAAAAAAAAAAAAACEBjw0a_L8M0F4xYND99dvahlrkpxODi9Wc9VzUvkhD0t{
    "v": "ACDC10JSON0005f2_",
    "d": "EIGWggWL2IHiUzj1P2YuPA0-Uh55LTIu14KTvVQGrfvT",
    "i": "ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
    "ri": "EAtQJEQMkkvlWxyfLbcLyv4kNeAI5Qsqe65vKIWnHKpx",
    "s": "EN6Oh5XSD5_q2Hgu-aqpdfbVepdpYpFlgz6zvJL5b_r5",
    "a": {
        "d": "EJJjtYa6D4LWe_fqtm1p78wz-8jNAzNX6aPDkrQcz27Q",
        "dt": "2023-11-13T17:41:37.710691+00:00",
        "ids": [
            "did:web:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:webs:did-webs-service%3a7676:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:web:example.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:web:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe",
            "did:webs:foo.com:ENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe"
        ]
    },
    "r": {
        "d": "EEVTx0jLLZDQq8a5bXrXgVP0JDP7j8iDym9Avfo8luLw",
        "aliasDesignation": {
            "l": "The issuer of this ACDC designates the identifiers in the ids field as the only allowed namespaced aliases of the issuer's AID."
        },
        "usageDisclaimer": {
            "l": "This attestation only asserts designated aliases of the controller of the AID, that the AID controlled namespaced alias has been designated by the controller. It does not assert that the controller of this AID has control over the infrastructure or anything else related to the namespace other than the included AID."
        },
        "issuanceDisclaimer": {
            "l": "All information in a valid and non-revoked alias designation assertion is accurate as of the date specified."
        },
        "termsOfUse": {
            "l": "Designated aliases of the AID must only be used in a manner consistent with the expressed intent of the AID controller."
        }
    }
}-VA0-FABENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe0AAAAAAAAAAAAAAAAAAAAAAAENro7uf0ePmiK3jdTo2YCdXLqW7z7xoP6qhhBou6gBLe-AABAADQOX208DAmZEPb2v0XXF0N6WgxOdOxB3AsCBJds_vbAr7v1PQBA4MWNsXc8unk5UykbB8j538XGkzLtujekvIP
```

:::

### Adding service endpoint roles in KERI

::: informative service endpoint

* A hypothetical new role that could be submitted to KERI, would be the
   DIDCommMessaging role. It could be specified

```json
{
  "service": [
      {
        "id":"#Bgoq68HCmYNUDgOz4Skvlu306o_NY-NrYuKAVhk3Zh9c",
        "type": "DIDCommMessaging", 
        "serviceEndpoint": "https://bar.example.com"
      },
      {
        "id":"#BuyRFMideczFZoapylLIyCjSdhtqVb31wZkRKvPfNqkw",
        "type": "KERIAgent", 
        "serviceEndpoint": {
          "tcp": "tcp://bar.example.com:5542",
          "https": "https://bar.example.com" 
        }
      }
  ]
}
```
::: 

## Annex

### KERI Fundamentals

::: informative KERI Fundamentals

[[ref: KERI]] is a protocol for managing
cryptographic keys, identifiers, and associated verifiable data structures.
KERI was first described in an
[academic paper](https://arxiv.org/abs/1907.02143), and its
[specification](https://github.com/trustoverip/kswg-keri-specification) is
currently incubated under [Trust Over IP Foundation](https://trustoverip.org/).
The open source community that develops KERI-related technologies can be
found at `https://github.com/WebOfTrust/keri`. This section outlines the
fundamentals and components of the KERI protocol that are related to the
`did:webs` method.

#### Autonomic Identifier (AID)

An [[ref: autonomic identifier]] is a globally-unique persistent
self-certifying [[ref: verifiable identifier]] that serves as the primary
root-of-trust of the KERI protocol. An AID is cryptographically bound to a
[[ref: KEL]] that
determines the evolution of its [[ref: key state]] using the
[[ref: pre-rotation]] mechanism. AIDs and the underlying KERI protocol, by
themselves, satisfy most of the
[properties](https://www.w3.org/TR/did-core/#design-goals) that DIDs require,
including decentralization, control, security, proof-based, and portability.
For example, DIDs that have the same AID component are considered
[equivalent](#equivalent-identifiers), allowing AIDs to be portable across
different DID methods such as `did:webs` and `did:keri`.

#### Key Event Log (KEL)

The binding between an [[ref: AID]] and its cryptographic keys is proved by
a data structure called a [[ref: key event log]] that allows the
[[ref: key states]] of the AID to evolve. For a `did:webs` DID, a KEL is
an essential component in the [[ref: KERI event stream]] that is used to
verify its authenticity.

A KEL is a hash-chain append-only log and can be considered a variant of
blockchain. However, a KEL differs from the traditional blockchain technology
in at least two important ways:

* It records the [[ref: key event]] history of a single AID with a single
   [[ref: controller]], instead of an arbitrarily large collection updated by
   other participants in the network. This makes a KEL memory-efficient, fast,
   cheap, and trivially scalable.
* It is fully self-certifying, meaning its correctness can be proved
   by direct inspection, without a distributed consensus algorithm or
   assumptions about trust in an external data source or its governance.

These properties allows a KEL to be published anywhere, without special
guarantees from its storage mechanism. For example, a KEL of an AID could be
published and migrated between different KERI-compatible blockchain networks
that use different DID methods. A KEL also records changes to key types and
cryptographic algorithms, providing the AID portability and adaptability to
multiple ecosystems throughout its lifecycle.

#### AID Derivation

The value of an [[ref: AID]] is derived from the first [[ref: key event]],
called the [[ref: inception event]], of a [[ref: KEL]]. The inception event
includes inital public key(s), called the _current_ key(s), that can be used
to control the AID. The cryptographic relationship between the AID and its
keys eliminates an early chain-of-custody risk that plagues many other DID
methods where an attacker uses compromised keys to create a DID without the
DID controller's knowledge. This derivation process is similar to techniques
used by `did:key`, `did:peer`, `did:sov`, and `did:v1`.

The simplest AIDs, called non-transferrable [[ref: direct mode]] AIDs, have
no additional input to the derivation function, and expose a degenerate KEL
that can hold only the inception event. This KEL is entirely derivable from
the AID itself, and thus requires no external data. Non-transferrable direct
mode AIDs are ideal for ephemeral use cases and are excellent analogs to
`did:key` and `did:peer` with `numalgo=0`. This is by no means not the only
option as KERI offers richer choices that are especially valuable if an AID
is intended to have a long lifespan.

#### Pre-rotation

Public keys in the [Verification Methods](#verification-methods) of a
`did:webs` DID can be changed via a mechanism called [[ref: pre-rotation]].
With pre-rotation, the inception event of the associated AID also includes
the hash(s) of the _next_ key(s) that can be used to change the
[[ref: key state]] of the AID. AIDs with one or more pre-rotated _next_ keys
are called _transferrable_ AIDs because their control can be transferred to
new keys. AIDs that do not use pre-rotation cannot change their keys and are
thus _non-transferrable_.

Pre-rotation has profound security benefits. If a malicious party steals the
_current_ private key for a transferrable AID, they only accomplish
_temporary_ mischief, because the already-existing KEL contains a commitment
to a future state. This prevents them from rotating the stolen AID's key to
an arbitrary value of their choosing. As soon as the AID controller suspects
a compromise, they can change the key state of the AID using the pre-rotated
_next_ key and locks the attacker out again.

#### Weighted Multisig

The [[ref: KERI]] protocol supports a weighted [[ref: multi-signature]] scheme that
allows for [conditional proof](#thresholds) in the
[Verification Methods](#verification-methods). A multisig [[ref: AID]]
distributes its control among multiple key holders. This includes simple
M-of-N rules such as "3 of 5 keys must sign to change the AID's
[[ref: key state]]". More sophisticated configurations are also supported:
"Acme Corp's AID is controlled by the keys of 4 of 7 board members, or by the
keys of the CEO and 2 board members, or by the keys of the CEO and the Chief
Counsel."

The security and recovery benefits of this feature are obvious when an AID
references organizational identity. However, even individuals can benefit
from this, when stakes are high. They simply store different keys on
different devices, and then configure how their devices constitute a
management quorum. This decreases risks from lost phones, for example.

#### Witnesses

In the [[ref: direct mode]], the [[ref: controller]] of an [[ref: AID]] is
responsible for the distribution of its [[ref: KEL]]. Since the controller
may not be highly available, the controller may designate additional
supporting infrastructure, called [[ref: witnesses]], for the
[[ref: indirect mode]] distribution of the [[ref: KELs]]. Witnesses may or
may not be under direct control of the AID's controller and could be
deployed on either centralized or decentralized architectures, including
blockchains. The witnesses are embedded in the AID's [[ref: key event]] and,
if included in the [[ref: inception event]], alter the AID value.

Unlike a blockchain with a distributed consensus mechanism, witnesses do not
coordinate or come to consensus with one another during an update to its
AID's [[ref: key event]] history. Hence, they need not be deeply trustworthy;
merely by existing, they improve trust. This is because anyone changing an
AID's key state or its set of witnesses, including the AID's legitimate
controller, has to report those changes to the witnesses that are currently
active, to produce a valid evolution of the KEL. The AID controller and all
witnesses thus hold one another accountable. Further, it becomes possible to
distinguish between duplicity and imperfect replication of key states.

#### Transaction Event Log (TEL)

The [[ref: KERI]] protocol supports a verifiable data structure, called the
[[ref: transaction event log]], that binds an [[ref: AID]] to non-repudiable
data that is deterministically bound to the [[ref: key event]] history in the
[[ref: KEL]]. Transactions that are recorded in a TEL may include things
like the issuance and revocation of verifiable credentials or the fact that
listeners on various service endpoints started or stopped. Like KELs, TELs are
self-certifying and may also be published by KERI [[ref: witnesses]] to enhance
discoverability and provide [[ref: watcher]] networks the ability to detect
duplicity.
For example, we demonstrate that in this spec in how we anchor
[[ref: designated aliases]] as
[verifiable data on a TEL](#verifiable-data-on-a-tel).

#### Web Independence

Although _this DID method depends on web technology, KERI itself does not_.
It's as easy to create AIDs on IOT devices as it is on the web. AIDs offer
the same features regardless of their origin and besides HTTP, they are
shareable over Lo-Ra, Bluetooth, NFC, Low Earth Orbit satellite protocols,
service buses on medical devices, and so forth. Thus, KERI's AIDs offer a
bridge between a web-centric DID method and lower-level IOT ecosystems.

#### Flexible Serialization

[[ref: KELs]] and [[ref: TELs]] of `did:webs` DIDs (i.e., AIDs) are included
in the [[ref: KERI event streams]] for verification of the DID documents.
The KERI event streams use [[ref: CESR]] for data serialization. Although CESR is a deep subject all
by itself, at a high level, it has two essential properties:

* **Content in CESR is self-describing and supports serialization as
   binary and text**: That is in [[ref: CESR]], _a digital signature on a
   CESR data structure is stable no matter which underlying serialization
   format is used_.  In effect it supports multiple popular serialization
   formats like JSON, CBOR, and MsgPack with room for many more. These
   formats can be freely mixed and combined in a CESR stream because of the
   self-describing nature of these individual CESR data structures.  The
   practical effect is that developers get the best of both worlds: they can
   produce and consume data as text to display and debug in a human-friendly
   form and they can store and transmit this data in its tersest form, _all
   without changing the signature on the data structures_.
* **Cryptographic primitives are structured into compact standard
   representations**: Cryptographic primitives such as keys, hashes, digests,
   sealed-boxes, signatures, etc... are structured strings with a
   recognizable data type prefix and a standard representation in the stream.
   This means they are very terse and there is no need for the variety of
   representation methods that create interoperability challenges in other
   DID methods (`publicKeyJwk` versus `publicKeyMultibase` versus other; see
   the verification material section of the [DID specification](#W3C-DID-CORE).

Despite this rich set of features, KERI imposes only light dependencies on
developers. The cryptography it uses is familiar and battle-hardened, exactly
what one would find in standard cryptography toolkits. For example, the
python implementation (keripy) only depends on the `pysodium`, `blake3`, and
`cryptography` python packages. Libraries for KERI exist in javascript, rust,
and python.

::: 



## Bibliography

### Normative section

<a id="DataIntegrityProof"></a>. W3C, [Data Integrity Proofs](https://www.w3.org/TR/vc-data-model/#data-integrity-proofs).

<a id="JSONSerialization"></a>. IETF, [RFC 7515 Section 3.2: JWS JSON Serialization](https://datatracker.ietf.org/doc/html/rfc7515#section-3.2).

<a id="MultiformatMultihash"></a>. Multiformats, [multihash](https://github.com/multiformats/multihash).

<a id="RFC1035"></a>. IETF, [RFC 1035: Domain Names - Implementation and Specification](https://www.rfc-editor.org/info/rfc1035).

<a id="RFC1123"></a>. IETF, [RFC 1123: Requirements for Internet Hosts - Application and Support](https://datatracker.ietf.org/doc/html/rfc1123).

<a id="RFC2181"></a>. IETF, [RFC 2181: Clarifications to the DNS Specification](https://www.rfc-editor.org/info/rfc2181).

#### Trust-over-IP

<a id="KSWG-ACDC"></a>. Trust over IP, [Authentic Chained Data Containers (ACDC) specification](https://trustoverip.github.io/kswg-acdc-specification/).

<a id="KSWG-CESR"></a>. Trust over IP, [Composable Event Streaming Representation (CESR)](https://trustoverip.github.io/kswg-cesr-specification/).

<a id="KSWG-CESR-PROOF"></a>. Trust over IP, [CESR Proof Signatures specification](https://trustoverip.github.io/tswg-cesr-proof-specification/draft-pfeairheller-cesr-proof.html).

<a id="KSWG-KERI"></a>. Trust over IP, [Key Event Receipt Infrastructure (KERI) specification](https://trustoverip.github.io/kswg-keri-specification/).

<a id="KSWG-ACDC-IPEX"></a>. Trust over IP, [ACDC IPEX validation](https://trustoverip.github.io/kswg-acdc-specification/#ipex-validation).

#### W3C

<a id="W3C-DID-WEB"></a>. W3C CCG, [did:web Method Specification](https://w3c-ccg.github.io/did-method-web/).

<a id="W3C-DID-CORE"></a>. W3C, [Decentralized Identifiers (DIDs) v1.0](https://www.w3.org/TR/did-1.0/).

<a id="W3C-DID-REGISTRIES"></a>. W3C, [DID Spec Registries](https://www.w3.org/TR/did-extensions/).

<a id="W3C-DID-PATH"></a>. W3C, [DID URL path](https://www.w3.org/TR/did-core/#path).

<a id="W3C-VCDM"></a>. W3C, [Verifiable Credentials Data Model](https://www.w3.org/TR/vc-data-model/).

<a id="W3C-VP"></a>. W3C, [Verifiable Presentations](https://www.w3.org/TR/vc-data-model/#presentations-0).

### Informative section

::: informative
<a id="DIDWEBS-REF-IMPL"></a>. GLEIF, [did:webs Reference Implementation](https://github.com/GLEIF-IT/did-webs-resolver) and its [getting started guide](https://github.com/GLEIF-IT/did-webs-resolver/blob/main/docs/getting_started.md).

<a id="OCA"></a>. Hyperledger Aries RFCs, [Overlay Capture Architecture](https://github.com/hyperledger/aries-rfcs/blob/main/features/0755-oca-for-aries/README.md).

<a id="RFC5895"></a>. IETF, [RFC 5895: Mapping Characters for IDNA2008](https://www.rfc-editor.org/info/rfc5895).

<a id="StatusList2021"></a>. W3C CCG, [VC Status List 2021](https://github.com/w3c-ccg/vc-status-list-2021).

<a id="UTS46"></a>. Unicode Consortium, [UTS #46: Unicode IDNA Compatibility Processing](https://unicode.org/reports/tr46/).

<a id="W3C-VC-RENDER"></a>. W3C CCG, [VC Rendering Methods specification](https://w3c-ccg.github.io/vc-render-method/).
:::
