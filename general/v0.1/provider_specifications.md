# Fediverse Auxiliary Service Providers: Fediverse Server Interaction

## 05: FASP Specifications

All FASP specifications MUST be based on this document and MUST NOT
change any of the requirements stated.

### Mandatory Content

Every FASP specification MUST include the following:

* An identifier for the specification
* A version number (see next section for details)
* Optional specification dependencies
* The names, identifiers and descriptions of the capabilities offered
* The FASP's HTTP API endpoints
* The HTTP API an instance needs to implement for the fediverse server
  to use (or a note that none is required)
* A summary of personally identifiable information or other data that
  might be considered sensitive that a FASP may receive

### Identifiers

Both the full specification and individual capabilities MUST have an
unique identifier suitable for use in URLs.

Identifiers are UTF-8 strings containing only lower-case characters and
no whitespace characters. It succinctly describes the specification or
capability in english using as few words as possible (ideally only a
single word). If more than one word is needed they MUST be separated by
an underline character (`_`).

Identifiers MUST be unique, which means they must not be defined in any
other provider specification.

Examples of valid identifiers:

* `debug`
* `discovery`
* `trends`
* `account_search`
* `media_storage`
* `spam_detection`

### Versioning

Every FASP specification MUST have a version number consisting of a
major and a minor number, delimited by a single `.` (dot).

Any change to a specification that runs the risk of being incompatible
to existing implementations MUST increase the major version number.

Clarifications, additions of examples etc. that are not expected to impact
existing implementations SHOULD increase the minor version number.

Formatting changes, spelling and grammar corrections MAY increase the
minor version number.

If only the details for a single capability change but the specification
includes several capabilities this means the version number would change
for all capabilities even for those that remain unchanged. If this
happens it might be a signal that the capabilities are not closely
related after all and should be extracted to their own separate
specifications.

### Specification Dependencies

Specifications MAY depend on other specifications. Some specifications
could include shared, overarching APIs that are useful in more than one
capability. In these cases other dependencies may simply state that they
depend on this. Dependencies MUST always refer to a specific version of
another specification.

If a specification declares a dependency on another, implementers MUST
also implement this other specification.

### Capabilities

Every FASP specification MAY define one or more capabilities.
Capabilities are units of functionality that a fediverse server admin
can enable or disable.

If a specification defines more than one capability, the capabilities
SHOULD be strongly related. This is the case for example when all
capabilities rely on some shared functionality or if an update to the
specification of one capability is expected to require an update of the
other as well.

For every capability defined, the specification MUST include a unique
identifier. See section "Identifiers" above for more details.

For every capability defined, the specification MUST also include a one
paragraph description in English that can be used by fediverse software
to explain the capabilities to fediverse server administrators. It MAY
include translations into other languages as well.

### HTTP API endpoints

All specified API endpoints, be it on FASP or fediverse servers, MUST
use paths relative to the base URL (see [Protocol
Basics](protocol_basics.md)).

When specifying URL paths or when giving examples of URL paths the base
path from the base URL MAY be omitted.

All relative paths MUST begin with the specification identifier as the
first segment.

The second segment MUST be the major version number prefixed with the
lower-case letter `v`.

Endpoints relating to a particular capability MUST use the capability
identifier as the third path segment.

FASP specifications MAY include both API endpoints that are common to
all capabilities and thus use paths that are only prefixed with the
specification identifier and API endpoints specific to one capability
only. These must also include the capability identifier.

For the first case, an overarching functionality used by more than one
capability, the pattern for paths looks like this:

```
/<specification identifier>/v<major version>/<descriptive path>
```

For the second case, functionality belonging to one specific capability,
the pattern looks like this:

```
/<specification identifier>/v<major version>/<capability identifier>/<descriptive path>
```

For example, a hypothetical `spam_detection` specification could have
different capabilities for classifying different types of content (post
text, accounts, images etc.) that use a shared vocabulary.

Getting the vocabulary via an endpoint would be a common functionality
not tied to any one of the capabilities and could thus result in the
following path:

```
/spam_detection/v2/vocabulary
```

And an endpoint that is tied to and only used for the
`image_classification` capability could have the following path:

```
/spam_detection/v2/image_classification/classification
```

### Privacy Policy Information

FASPs may receive personally identifiable information and other data
that some people might consider sensitive. This must then become part of
a fediverse server's privacy policy to inform users of what data is being
processed, by whom, and how.

Since FASP specifications define exactly what data the FASP can
receive, they MUST include a summary noting which of this data is to be
considered sensitive and why it needs to be processed by the FASP.

This summary MUST make it easy to copy this directly into the privacy
policies of fediverse servers.
