# Fediverse Auxiliary Service Providers: Fediverse Server Interaction

## 05: FASP Specifications

All FASP specifications MUST be based on this document and MUST NOT
change any of the requirements stated.

### Mandatory Content

Every FASP specification MUST include the following:

* A version number (see next section for details)
* The names and descriptions of the capabilities offered
* The FASP's HTTP API endpoints
* The HTTP API an instance needs to implement for the fediverse server to use
  (or a note that none is required)
* A summary of personally identifiable information or other data that
  might be considered sensitive that a FASP may receive

### Versioning

Every FASP specification MUST have a version number consisting of a major and
a minor number, delimited by a single `.` (dot).

Any change to a specification that runs the risk of being incompatible
to existing implementations MUST increase the major version number.

Clarifications, additions of examples etc. that are not expected to impact
existing implementations SHOULD increase the minor version number.

Formatting changes, spelling and grammar corrections MAY increase the
minor version number.

### Capabilities

Every FASP specification MUST define at least one capability. It MAY
define more than one capability, but only if the capabilites are
strongly related.

For every capability defined, the specification MUST include a unique
identifier. The identifier is a UTF-8 string containing only lower-case
characters and no whitespace characters. It succinctly describes the
capability in english using as few words as possible (ideally only a
single word). If more than one word is needed they MUST be separated by
an underline character (`_`).

Identifiers MUST be unique, which means they must not be defined in any
other provider specification.

Examples of capability identifiers:

* `trends`
* `account_search`
* `link_previews`
* `media_storage`
* `spam_detection`

For every capability defined, the specification MUST also include a one
paragraph description in English that can be used by fediverse software
to explain the capabilities to fediverse server administrators. It MAY include
translations into other languages as well.

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
