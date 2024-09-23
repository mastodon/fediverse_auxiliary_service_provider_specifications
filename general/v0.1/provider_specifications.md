# Fediverse Auxiliary Service Providers: Provider-Instance Interaction

## 05: Provider specifications

All provider specifications MUST be based on this one and MUST NOT
change any of the requirements stated above.

### Mandatory Contents

Every specification MUST include the following:

* A version number (see next section for details)
* The names and descriptions of the provided capabilities
* The provider's HTTP API endpoints
* The HTTP API an instance needs to implement for the provider to use
  (or a note that none is required)
* The OAuth 2.0 scopes used
* A summary of personally identifiable information or other data that
  might be considered sensitive that a provider may receive

### Versioning

Every specification MUST have a version number consisting of a major and
a minor number, delimited by a single `.` (dot).

Any change to a specification that runs the risk of being incompatible
to existing implementations MUST increase the major version number.

Clarifications, additions of examples etc. that should not impact
existing implementations SHOULD increase the minor version number.

Formatting changes, spelling and grammar corrections MAY increase the
minor version number.

### Capabilities

Every provider specification MUST define at least one capability. It MAY
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
paragraph description in english that can be used by fediverse software
to explain the capabilities to instance administrators. It MAY include
translations into other languages as well. 

### OAuth 2.0 Scopes

Provider specifications MUST define the scopes needed to access the
provider's API endpoints. There SHOULD be at least one scope. Scope
names MUST begin with the capability identifier to not collide with
scopes defined in other provider specifications.

In the simplest case, there is exactly one scope and the scope's name is
the capability identifier. Finer grained scopes MAY be defined and MUST
use a colon (`:`) character to seperate the capability identifier from
any additional characters.

Examples scope names on the provider side:

* `debug`
* `trends`
* `account_search:write`
* `media_storage:videos:read`

If a provider specification also defines API endpoint on the instances
side then the same rules apply with one addition:

To prevent collisions with existing scope names an instance's fediverse
software might already use, all scope names MUST be prefixed with
`aux:`.

Example scope names on the instance side:

* `aux:debug`
* `aux:account_search:write`

### Privacy Policy information

Providers may receive personally identifiable information and other data
that some people might consider sensitive. This must then become part of
an instance's privacy policy to inform users of what data is being
processed and how.

Since provider specifications define exactly what data a provider can
receive, they MUST include a summary noting which of this data is to be
considered sensitive from a data protection point of view and why it
needs to be processed by the provider.

This summary MUST make it easy to copy this directly into the privacy
policies of instances.
