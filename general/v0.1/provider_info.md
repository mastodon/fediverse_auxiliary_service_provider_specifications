# Fediverse Auxiliary Service Providers: Provider-Instance Interaction

## 04: Provider Info

Every provider must offer a provider info API endpoint that can be queried to obtain information about the provider.

This endpoint can be queries using a `GET` call to `/provider_info`.

Example call:

```http
GET /provider_info
```

Example response:

```json
{
  "name": "Example provider",
  "privacy_policy": [
    {
      "url": "https://provider.example.com/privacy.html",
      "language": "en"
    }
  ],
  "capabilities": [
    {
      "id": "trends",
      "version": "1.0"
    },
    {
      "id": "account_search",
      "version": "1.0"
    }
  ],
  "sign_in_url": "https://provider.example.com/sign_in"
}
```

The provider info MUST contain the keys `name`, `privacy_policy` and
`capabilities`. The key `sign_in_url` MUST be present if the provider offers a
way for an instance administrator to sign in (e.g. to edit settings, show
statistics etc.). Otherwise it MAY be absent.

The values are defined as follows:

* `name`: This MUST be a non-empty string containing the name of this provider.
  This is the name of this installation of the provider software. It will be
  displayed to instance administrators and SHOULD be as descriptive as possible
  for them to identify the provider.
* `privacy_policy`: This MUST be an array. In case the provider does not
  receive any kind of personally identifiable information or other data that
  might be considered sensitive this MAY be empty. Otherwise it MUST contain
  one objects containing the keys `url` and `language` for each language the
  privacy policy is available in:
    * `url`: URL of the publicly available privacy policy. Instance operators
      may link to this in their own privacy policy.
    * `language`: A language code representing the language of the privacy
      policy.
* `capabilities`: This MUST be an array including one object for every
  capability the provider implements. These objects contain the keys `id` and
  `version`:
    * `id`: This is the identifier of the capability. See provider
      specifications for the list of capabilites and their identifiers.
    * `version`: The version of the specification for this capability that this
      provider supports.
* `sign_in_url`: If present it MUST contain a string representing the URL where
  an instance administrator can manually sign in to the provider.

---

Next: [05: Provider Specifications](provider_specifications.md)
