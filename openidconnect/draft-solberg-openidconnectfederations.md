# OpenID Connect Federations

* Identifier: draft-solberg-openidconnectfederations
* Editor: Andreas Åkre Solberg, UNINETT AS, <mailto:andreas.solberg@uninett.no>
* Version 0.1. *Last edited July 6th 2011.*

This document is currently just some notes, thinking loud about the possibility of building a federation on top of OpenID Connect. If it turns out to be a good idea, these notes may migrate into a spec.

The reason why I even bother to think about this is that I think it is really pity that Federations and the OpenID community is two split worlds. The users (Service Providers) are the ones that are hurt by this, and I don't like that. If there is only a slight possibility that these worlds could meet, I think it is worth exploring.


Useful references:

* OpenID Connect: <http://openidconnect.com/>
* OAuth 2.0: <http://tools.ietf.org/html/draft-ietf-oauth-v2>


[WebFinger]: http://hueniverse.com/2009/08/introducing-webfinger/
[XRD]: http://docs.oasis-open.org/xri/xrd/v1.0/xrd-1.0.html
[JRD]: http://hueniverse.com/2010/05/jrd-the-other-resource-descriptor/
[JSON Simple Sign]: http://jsonenc.info/jss/1.0/
[host-meta]: http://hueniverse.com/2009/11/host-meta-aka-site-meta-and-well-known-uris/
[Magic Signatures]: http://salmon-protocol.googlecode.com/svn/trunk/draft-panzer-magicsig-01.html
[Magic JSON Envelope]: http://salmon-protocol.googlecode.com/svn/trunk/draft-panzer-magicsig-01.html#anchor5

[OpenID Connect Discovery]: http://openidconnect.com/#discovery
[OpenID Connect Response]: http://openidconnect.com/#response


<!-- {{TOC}} -->


## Terminology

I use the Service Provider and Identity Provider terms from the *federation world* on purpose.



## Metadata

Examples given in the following sections shows metadata for a single entity. A complete metadata document will usually be represented as a list of entity metadata for either Identity Providers or Service Providers, wrapped in a Metadata Container. Metadata is always represented in a JSON object.

The Metadata Container is discussed in the *Federation Trust* section.

*TODO: Consider wrap the metadata in [XRD][] / [JRD][].*




### Identity Provider Metadata

The following properties may be associated with the Identity Provider:


**Generic information**

`title`
: A good name (in default language) of the Identity Provider. Should be short and descriptive.

`descr`
: A description. Not more than one sentence.

`country`
: Two-letter code in capital letters for country of the Identity Provider. The value `_all_` means that the entity is associated with all countries.

`geo`
: A list of geographic coordinates of the Identity Provider.

`keywords`
: A list of searchable keywords associated with the Identity Provider


**OAuth specific information**

`oauth.endpoints.authorization`
: OAuth Authorization Endpoint

`oauth.endpoints.token`
: OAuth Token Endpoint

**OpenID Connect information**

`connect.endpoints.association`
: The association endpoint, for getting a temporary consumer key/secret. Discussed later.

`connect.endpoints.userinformation`
: The user information endpoint as described in [OpenID Connect][].


**Federation properties**


`federation.domains`
: A list of domains that this Identity Provider is allowed to assert user information about. The consumer will match this list with the `domain` property in the [OpenID Connect Response][].

`federation.loa`
: A list of Level of Assurance profiles that this Identity Provider fulfills according to the federation third party. Each level is identified by a URI.

`federation.endpoints.globallogout`
: The global logout endpoint, accepting logout requests as described in the section *Global Logout*

<!-- 

**Trust configuration**

The metadata data format may be used in a number of places. When the metadata is used for local configuration, the document may include a client key / secret for simplicity.

`oauth.consumer_key`
: OAuth Authorization Endpoint

`oauth.consumer_secret`
: OAuth Token Endpoint

**These parameters may be useful for the generic metadata format, but will not be used with OpenID Connect Federations**

-->





**Example metadata:**

	{
		title: 'Norwegian University of Science and Technology',
		descr: 'The Norwegian University of Science and Technology (NTNU) is Norway’s primary institution for educating the nation's future engineers and scientists.',
		keywords: ['NTNU', 'NUST', 'Trondheim'],
		country: 'NO',
		geo: [{lat: 63.4171494, lon: 10.40447}],
		
		oauth.endpoints.authorization: 'https://ntnu.no/oauth/auth',
		oauth.endpoints.token:  'https://ntnu.no/oauth/token',
		connect.endpoints.association:  'https://ntnu.no/oauth/assoc',
		connect.endpoints.userinformation:  'https://ntnu.no/oauth/userinfo',
		federation.domains: ['ntnu.no'],
		federation.loa: ['urn:oasis:names:tc:SAML:2.0:post:ac:classes:nist-800-63:v1-0-2:2'],
		federation.endpoints.globallogout: 'https://ntnu.no/oauth/logout'
		
	}





### Service Provider Metadata


**Generic information**

`title`
: A good name (in default language) of the Identity Provider. Should be short and descriptive.

`descr`
: A description. Not more than one sentence.

`url`
: The URL to access the service

**OAuth specific information**


`oauth.endpoints.redirection`
: OAuth Redirection Endpoint


**Federation Trust Configuration**

`federation.public_key`
: A list of public keys for the Service Provider. Should be represented using *The Magic Envelope Compact Serialization* [Magic Signatures][].

`federation.supported_algs`
: A list of supported algorithms. 

`federation.requested_attributes`
: A list of attributes that the consumer would like to have to function normally.

`federation.endpoints.globallogout`
: The global logout endpoint, accepting logout requests as described in the section *Global Logout*

**Example metadata:**

	{
		title: 'Foodle',
		descr: 'The supercool and simple webbased application for simple polls and surveys.',
		url: 'https://foodl.org',
		oauth.endpoints.redirection: 'https://foodl.org/oauth/callback',
		federation.public_key: [
			'RSA.mVgY8RN6URBTstndvmUUPb4UZTdwvwmddSKE5z_jvKUEK6yk1u3rrC9yN8k6FilGj9K0eeUPe2hf4Pj-5CmHww.AQAB',
		].
		federation.public_key: [
			'RSA-SHA256', 'RSA-SHA1'
		],
		federation.requested_attributes: [
			'user_id', 'email', 'display_name', 'edu_eduPersonAffiliation', 'edu_norEduOrgAcronym'
		],
		federation.endpoints.globallogout: 'https://foodl.org/oauth/logout'
	}



### Federation Metadata

`title`
: Title of the circle of trust. For easier recognition in logs and configuration. 

`federation.public_key`
: A list of public keys for the Federation operator. Should be represented using *The Magic Envelope Compact Serialization* [[Magic Signatures][]].

`federation.supported_algs`
: A list of supported algorithms.

`endpoint.trusted-providers`
: An endpoint listing trusted Identity Providers.

`endpoint.trusted-consumers`
: An endpoint listing trusted Service Providers.


Example:

	{
		title: 'eduGAIN',
		endpoint.trusted-providers: 'http://federation.org/trusted-providers',
		endpoint.trusted-consumers: 'http://federation.org/trusted-consumers',
		federation.public_key: [
			'RSA.mVgY8RN6URBTstndvmUUPb4UZTdwvwmddSKE5z_jvKUEK6yk1u3rrC9yN8k6FilGj9K0eeUPe2hf4Pj-5CmHww.AQAB',
		],
		federation.supported_algs: [
			'RSA-SHA256', 'RSA-SHA1'
		]
	}


## Federation Trust

Let us introduce a third party asserting a set of trusted Identity Providers and Service Providers (circle of trust).

To establish a federation, there will be created two documents:

* Trusted Service Providers, and
* Trusted Identity Providers

served over plain HTTP on each endpoint.

Each document is a list of Service Provider (or Identity Provider respectively) metadata objects wrapped in a *Metadata Container*, and signed using [[Magic Signatures][]].

The Metadata Container looks like this:

	{
		'expires': 1309945835,
		'lastUpdated': 1309942321,		
		'serial': 1039,
		'entries': [
			{ ...consumer or provider... },
			{ ...consumer or provider... },
			...
		]
	}

The properties are:

`expires`
: When does the signed federation metadata document expires in a UTC unix timestamp. After this time, the content SHOULD NOT be trusted.

`lastUpdated`
: Timestamp for latest change of the document content.

`serial`
: Sequential increasing serial number for the document. Should be incremented on each change of the content.

`entries`
: A list of Service Provider metadata objects or Identity Provider metadata objects.


After the document is signed and wrapped in a [Magic JSON Envelope][] is may look like this:

	{
		data: "afb98af7b87af687f6876ba87f687a5b76a5..."
		data_type: "application/json",
		encoding: "base64url",
		alg: "RSA-SHA256",
		sigs: [{
			value: "EvGSD2vi8qYcveHnb-rrlok07qnCXjn8YSeCDDXlbhILSabgvNsPpbe76up8w63i2fWHvLKJzeGLKfyHg8ZomQ",
			keyhash: "4k8ikoyC2Xh+8BiIeQ+ob7Hcd2J7/Vj3uM61dy9iRMI="
		}]
	}



## Dynamic Association

The consumer sends a POST request to the `federation.endpoints.association` endpoint in the Identity Provider metadata.

The body of the request will be a Dynamic Association Request encoded in JSON and signed using [Magic Signatures][], using the key associated with one of the public keys listed in the metadata of the consumer.

The request MUST contain the following properties:

`type`
: This property MUST be set to `client_associate`.

`redirection_url`
: This value MUST match the `oauth.endpoints.redirection` in the consumer metadata.

Here are some additional properties that may be added

`federation.loa`
: This property is optional, and may be useful if the Identity Provider offers more than one loa value in the Identity Provider metadata. The Identity Provider may use this to select authentication methods with the user. The value may in example influence whether the user will need to use two-factor authentication or not.

`federation.sign`
: This property SHOULD be included and set to `true`. It indicates to the Identity Provider that it wants the association response signed. The Identity Provider MUST NOT sign the response unless this parameter is set to true in the request, to ensure compatibility with plain OpenID Connect.

Here is an example of the `Dynamic Association Request`:

	{
		type: 'client_associate'
		redirect_uri: 'https%3A%2F%2Ffoodl.org%2Foauth%2Fcallback',
		sign: true
	}

**NOTE: This proposal involves using an `application/json` type of the request body. The current OpenID Connect spec differs by using `application/x-www-form-urlencoded`.**


The Identity Provider then tries to match the `redirect_uri` from the `oauth.endpoints.redirection` properties found in the trusted consumer metadata.


If the signature was valid, and the Identity Provider trust the consumer, the Identity Provider will respond with a new consumer key / secret pair, like this:

	{
		client_id: 'consumer',
		client_secret: 'key',
		expires_in: 3600,
		flows_supported: ['web_server', 'user_agent'],
		user_endpoint_url: 'https://foodl.org/oauth/callback',
	}

If the request contains the sign request parameter and it is set to `true` the Identity Provider SHOULD sign the response. The response is signed using [Magic Signatures][], using the key associated with one of the public keys listed in the metadata of the Identity Provider.


In some cases the Identity Provider trusts the consumer, but is not willing to release all the attributes requested in the consumer metadata. If this is the situation and the Identity Provider will still release at least one of the requested attributes, the Identity Provider MUST return this additional property in the response:

`attributes`
: A list of attributes that the Identity Provider will release to the Service Provider. This will be a subset of the attributes in Service Provider metadata.

If the `attributes` property of the response is not present, the consumer can expect all the requested attributes to be released.

If the Provider does not accept the Service Provider sufficiently to allow users to login, it should fail at this point, returning an error instead:

	{
		'error_type': 'noaccess',
		'error_descr': 'The provider is awaiting the Identity Provider operators to \
			manually and explicitly opt-in for access to this consumer. This consumer was discovered \
			in metadata 9 days ago, if you have questions about the acceptance procedures for new \
			consumers, contact federation-helpdesk@ntnu.no'
	}





## Global Logout

A global logout endpoint will be able to process a HTTP GET request with the following query string parameters:

`consumer_key`
: The consumer_key that needs to be invalidated.

The endpoint MUST then return a status JSON object like this:

	{
		status-self: 'ok',
		status-propagation: 'na'
	}

There are two properties that always MUST be present in the response:

`status-self`
: Did the owner of the global logout endpoint successfully managed to logout the user (invalidate the consumer key).

`status-propagation`
: If the target has derived the session from another party, or is the authoritative source of the session for other parties, the target should try to propagate the logout to those external parties. This property indicate whether that was successful or not. A value of `ok` means that all external parties returned success, a value of `partial` means that at least one of the external parties returned success but not all, and `fail` means that none of the external parties did not return success. `na` means that the target have not associated the local session with any external parties (except from association with the requestor).

If the provider wants to include a more descriptive error message in the response it may include an `error_descr` property with a human readable error message.

Both consumers and providers should have global logout endpoints. If a consumer or provider does not properly support global logout as described in this section, the entity MUST NOT include a global logout endpoint in the metadata.

The global logout endpoint MUST NOT require cookies to be present; allowing an entity to enforce logout using a backchannel HTTP call.


<!-- 

## Notes


TODO List:

* Metadata
* Federation trust through a third party
* Dynamic association bootstrapped from the federation trust
* Discovery (or Directory of Providers)
	* Compatibility with [WebFinger][]
	* Compatibility with [OpenID Connect Discovery][]
	* Consider [host-meta][]
* Automatic exchange of bi-lateral trust relationships within the federation.
* Automated negotiation of attribute release




Important:

* Re-introduce some crypto in OAuth.
* Add federation as a layer on top of OAuth 2.0, without modifying anything on the base layer. This will ensure simple deployment of Consumers that would like to both connect to federations and open providers.
* Keep it simple.
* Central component needs to be super light weight.


-->




