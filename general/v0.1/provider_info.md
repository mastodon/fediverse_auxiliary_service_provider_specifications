# Fediverse Auxiliary Service Providers: Fediverse Server Interaction

## 04: FASP Information

Every FASP must offer a FASP information API endpoint that can be queried to obtain information about the FASP.

This endpoint can be queried using a `GET` call to `/provider_info`.

Example call:

```http
GET /provider_info
```

Example response:

```json
{
  "name": "Example FASP",
  "privacyPolicy": [
    {
      "url": "https://fasp.example.com/privacy.html",
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
  "signInUrl": "https://fasp.example.com/sign_in",
  "contactEmail": "support@fasp.example.com",
  "fediverseAccount": "@fasp@fedi.example.com"
}
```

The FASP info MUST contain the keys `name`, `privacyPolicy` and
`capabilities`. The key `signInUrl` MUST be present if the FASP offers a
way for a fediverse server administrator to sign in (e.g. to edit settings, show
statistics etc.). Otherwise it MAY be absent.

The values are defined as follows:

* `name`: This MUST be a non-empty string containing the name of this FASP.
  This is the name of this installation of the FASP software. It will be
  displayed to fediverse server administrators and SHOULD be as descriptive as possible
  for them to identify the FASP.
* `privacyPolicy`: This MUST be an array. In case the provider does not
  receive any kind of personally identifiable information or other data that
  might be considered sensitive this MAY be empty. Otherwise it MUST contain
  one object containing the keys `url` and `language` for each language the
  privacy policy is available in:
    * `url`: URL of the publicly available privacy policy. Fediverse server administrators
      may link to this in their own privacy policy.
    * `language`: An ISO-639-1 language code representing the language of the privacy
      policy.
* `capabilities`: This MUST be an array including one object for every
  capability the FASP implements. These objects contain the keys `id` and
  `version`:
    * `id`: This is the identifier of the capability. See FASP
      specifications for the list of capabilites and their identifiers.
    * `version`: The version of the specification for this capability that this
      FASP supports.
* `signInUrl`: If present it MUST contain a string representing the URL where
  a fediverse server administrator can sign in to the FASP.
* `contactEmail`: MAY be optionally included to communicate an email
  address to contact the FASP's operator(s).
* `fediverseAccount`: MAY be optionally included to point to a
  fediverse account that can be followed to receive updates about the
  FASP.

---

Next: [05: Provider Specifications](provider_specifications.md)
