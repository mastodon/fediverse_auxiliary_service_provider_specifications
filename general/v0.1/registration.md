# Fediverse Auxiliary Service Providers: Fediverse Server Interaction

## 03: Registration

### Registering with a FASP

When an administrator of a fediverse server software decides to start using a
FASP, they MAY be required to register with the provider. Every FASP
MAY have different requirements when it comes to fediverse server
registration. Different technical,
organisational or legal requirements may apply. Thus, this document does
not impose any hard requirements on that process, except for the end
result.

A FASP SHOULD document the process a fediverse server should use to register,
even when FASP registration is closed or by invitation only.

A FASP SHOULD list its capabilities and MAY name fediverse software
that is known to be compatible.

A FASP MAY provide a web-based form to register. Please see below
for an example.

During registration the provider MAY request data from the fediverse server
administrator. This MAY include but is not limited to the following:

* Email address and/or other contact data of the adminstrator
* A password of other means of authentication, so the administrator can
  sign-in again later
* URL of the fediverse server(s) to be registered
* Acceptance of terms of service, data processing agreement, and/or privacy policy

A successful registration results in the FASP creating an OAuth 2.0
application for the fediverse server, including a client identifier (ID) and
secret. FASP MUST grant the application all the scopes defined here and in
the FASP specifications of the capabilities the FASP advertises
to the fediverse server (see section [04: Provider Info](provider_info.md)).
FASP specifications MAY define exceptions to this rule.

After registration, FASP MUST present the fediverse server
administrator a registration token that can be copied into the fediverse server configuration.

The registration token is a JSON Web Token (JWT) that MUST include the
registered claims `iss` and `sub` and the private claim `sec`. It MAY
contain the registered claim `exp`.

These claims MUST contain the following:

* `iss`: The base URI for API requests to the FASP.
* `sub`: The OAuth 2.0 client identifier
* `sec`: The OAuth 2.0 client secret
* `exp`: An optional expiration time

An example payload:

```
{
  "iss": "https://fasp.example.com",
  "sub": "b2ks6vm8p23w",
  "sec": "H4bo9pER2Ww4MlPs2Rf",
  "exp": 1726498179
}
```

FASPs SHOULD offer a way to re-generate the OAuth credentials and
this token in case it gets lost or a fediverse server needs to be reinstalled
from scratch.

The following is a sketch of how this may look in the abstract:

Step 1: A fediverse server admin is presented with a registration form

![A bare-bones sign-up form asking for email, instance URL and acceptance of terms of service](../../images/instance_sign_up.svg)

Step 2: Upon successful registration, the registration token is
displayed

![A webpage displaying a registration token with a button to copy to clipboard](../../images/instance_sign_up_success.svg)

## Adding a FASP to a Fediverse Server

The option to add a new FASP SHOULD be part of the user interface
for administrative settings that a fediverse software already has.

To add a new FASP, the fediverse server adminstrator MUST enter the
registration token obtained in the previous step. The fediverse
software MUST validate the token before proceeding.

This validation MUST include:

* Ensuring the presence of the `iss`, `sub` and `sec` claims
* Ensuring that `iss` is a valid HTTP(S) URI
* If `exp` is present, ensure the date it represents is not in the past

![A bare-bones web form to add a provider with a textarea for the registration token](../../images/add_provider.svg)

After successful submission of this data, the fediverse software MUST
persist it and initiate an OAuth 2.0 "client credentials" flow to
obtain an access token. This token can then be used to authenticate
subsequent API calls.

If authorization was successful the instance MUST create and persist an
OAuth 2.0 application representing the FASP, including a client ID
and a secret.

It MUST communicate its base URL, the generated client ID and secret to
the FASP using the client credentials API endpoint.

Example request:

OAuth 2.0 scope: `setup`

```http
POST /client_credentials
```

Example request body:

```json
{
  "baseURL": "https://fediverse-server.example.com",
  "clientId": "kOP2Bc7oP31zM",
  "clientSecret": "pl5z4ciwfEP194MwWziP"
}
```

The FASP MUST respond with an HTTP status code `201` (Created)  and
persist the ID, secret and base URL it received.

Until this ID, secret and base URL have been communicated successfully,
FASP MAY refuse to accept API requests and respond with the HTTP
status code `424` (Failed Dependency) instead.

Fediverse software SHOULD catch this and retry sending the data in this
case.

### Selecting Capabilities

FASPs might implement any number of 
specificatons. As a last step in the setup process the
fediverse server administrator needs to select which capabilities of the
FASP they want to use.

In order to display available capabilities the fediverse server MUST
call the FASP info API endpoint (see
[04: Provider Info](provider_info.md) for a detailed description).

The response includes a list of capability identifiers (see
[05: Provider Specification](provider_specifications.md) for details)
and supported version numbers.

The fediverse software MUST present the administrator with the 
capabilities that the FASP supports.


![A web form on the instance that allows to select capabilities that both parties support](../../images/select_capabilities.svg)

---

Next: [04: Provider Info](provider_info.md)
