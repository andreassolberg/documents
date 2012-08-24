# OpenID Connect Federations

OpenID Connect does not have built-in mechanisms to support Identity Federations. This document presents some ideas around how to build a federation trust chain for OpenID Connect, to support the Identity Federation use cases.


The first typical scenario is to provide a dynamic set of trusted OpenID Connect providers that may be exposed to a specific OpenID Consumer.

The OpenID Consumer first configures a local *Trust Anchor Document*, that refers to specific providers and/or trusted third parties.





## OpenID Connect Trust Anchor Document

An *OpenID Connect Trust Anchor Document* (TAD) may be both stored in local configuration or retrieved via HTTP.

A TAD in an unpacked version is a JSON document, and when not stored locally it must always be encoded as a signed [JSON Web Token (JWT)][JWT].

The locally configured TAD typically defines the root of the trusted entities, and contains the magic issuer id `root`. An empty root TAD that trusts nobody, looks like this:

	{
		"iss": "root",
		"entities": [
		]
	}

The `entities` claim is an array of trusted entities. An entity may be:

* an OpenID Connect Provider
* an OpenID conncet Consumer
* a remote Trusted Third Party TAD provider

The entity type is defined in the `type` property.


### OpenID Connect Provider

Here is an example of a provider entity:

	{
		"type": "provider",
		"iss": "http://idp.feide.no",
		"jwk": {
			"keys": [
				{
					"alg": "RSA",
					"mod": "0vx7agoebGcQSuuPiLJXZptN9nndrQmbXEps2ai...",
					"exp": "AQAB",
					"use": "sig"
					"kid": "2011-04-29"
				} 
			]
		}
	}

The configuration of an Connect Provider MUST at least contain:

* type set to `provider`
* iss set to the issuer of the provider
* an embeeded `jwk`



An OpenID Consumer configured to trust such a provider, must follow the following step to validate the trust chain:

Say that the Consumer is about start authentication with the provider `http://idp.feide.no`. First it would need to traverse the trust chain to find this provider.

Next, it should resolve the configuration of the provider using [OpenID Connect Discovery][].

Now, perform the following validation of the discovery response:

* Reject the response if the `issuer` does not match the `iss` in the TAD.
* Reject the response if it does not contain `jwk` or `jwk_url`
* Resolve the `jwk_url` if used
* Reject the response if none of the `kid` values in the resolved *JWK* match any of the keys in the TAD.

Now, replace the JWKs resolved from the discovery with the ones from the TAD before further processing. Only include the ones with a matching `kid` value in both the TAD and the JWT resolved from discovery.

* TODO: Add support for encryption
* TODO: Add support for x509 in addition to JWT.


### OpenID Connect Consumer

TBD. Currently the focus is on providing a trust anchor chain of which providers to trust (for consumers).

### Trusted Third Party TAD Provider

By referring to a remotely located TAD, a dynamic trust chain may be created. Typically this can be used to configure trust of a federation, or a federation of federations.

	{
		"type": "tad",
		"iss": "http://edugain.org",
		"href": "http://edugain.org/trusted_federations",
		"refresh_interval": 3600,
		"jwk": {
			"keys": [
				{
					"alg": "RSA",
					"mod": "0vx7agoebGcQSuuPiLJXZptN9nndrQmbXEps2ai...",
					"exp": "AQAB",
					"use": "sig"
					"kid": "2011-04-29"
				} 
			]
		}
	}

When resolving a remote TAD provider, the client need to retrieve the document using HTTP from the URL in the `href` property using the *Trust Anchor Document Protocol* described below.

The `refresh_interval` gives a hint of reccomended interval to refetch the document in seconds. This is typically more often that the validity period of the docuement it self, and is sufficiently often to quickly allow newly registered providers to interop.

The obtained document will be a signed JWT. Before accepting the remotely obtained TAD, the following validation checks need to be performed:

* The `iss` of the obtained TAD MUST match the `iss` property in reference.
* Validate the JWT signature


## UI Metadata

In a federated environment the total set of providers is limited, and may be presented in a list rather than through regular OpenID Discovery. UI Metadata properties may be added to provider and consumer entity configurations.

All UI prpoerties are prefixed '`ui.`'. Here are the list of properties:


ui.title
: A good name (in default language) of the Login Provider. Should be short and descriptive.

ui.descr
: A description. Not more than one sentence.

ui.country
: Two-letter code in captial letters for country of the login provider. The value `_all_` means that the entity is associated with all countries.

ui.geo
: Geographic coordinates of the entity. It is a list of one or more coordinates.

ui.weight
: Weighting may be used to sort entities in a UI list. If not set, default is 0. If below 0, entity will pop up. If below -50, entity will be hidden in discovery UI.

Example of a provider with UI info,

	{
		"type": "provider",
		"iss": "https://uio.no",
		"jwk": {...},
		"ui.title": "University of Oslo",
		"ui.country": "NO",
		"ui.geo": [{
				lat: 63.8333,
				lon: 20.25
		}]
	}





## Restrictions

In a TAD docucent you may put restrictions on the children of a node, to restrict the delegated trust to remote third parties, or restrict the valid assertions of a Provider.

The `restrictions` property may be included in any entity config:

	{
		"type": "tad",
		"iss": "http://edugain.org",
		"href": "http://edugain.org/trusted_federations",
		"jwk": ...,
		"restrictions": {
			...
		}
	
	}

A restrictions applied on a TAD entity reference, will be applied to all entities loaded from that. The restrictions will cummulatively follow subsequent TAD references as well.




Restrict valid issuer identifiers to only those prefixed `https://edugain.org/providers/`.

	"restrictions": {
		"issuer": {
			"type": "prefix",
			"value": "https://edugain.org/providers/"
		}
	}

Restrict valid issuers to only norwegian realms.

	"restrictions": {
		"issuer": {
			"type": "match",
			"value": "https://*.no/*"
		}
	}

Restrict to list of valid issuers:
	
	"restrictions": {
		"issuer": {
			"type": "oneof",
			"value": ["https://consumer1.org", "https://consumer2.org", "https://consumer3.org"]
		}
	}

Assertions from these providers are restricted to only claim `arcs` of these specific values:

	"restrictions": {
		"arcs": ["1", "http://id.incommon.org/assurance/bronze"]
	}

Providers are limited to only trustworthly provide these attributes:

	"restrictions": {
		"userinfo": {
			"schema": "openid",
			"claims": ["user_id", "name", "email"]
		}
	}




## Trust Anchor Document Protocol

The client will perform a normal HTTP GET operation towards the `href` of the remote TAD entity.

	GET /trusted_federations HTTP/1.1
	Host: edugain.org
	Accept: application/jwt

	HTTP/1.0 200 OK
	Date: Fri, 31 Dec 2012 23:59:59 GMT
	Content-Type: text/html
	Content-Length: application/jwt


	eyJ0eXAiOiJKV1QiLA0KICJhbGciOiJIUzI1...


The client shuold cache the response. The validity period is expressed in the JWT `exp` property.

The response should be validated, and the client may perform subsequent requests to referred remote TADs in this document.

The client should implement loop detection.

The client should implement HTTP chache headers as described in RFC...


[OpenID Connect Discovery]: http://openid.net/specs/openid-connect-discovery-1_0.html
[JWT]: http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html





