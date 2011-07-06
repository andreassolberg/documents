# OpenID Connect Federations

* Identifyer: draft-solberg-openidconnectfederations
* Editor: Andreas Åkre Solberg, UNINETT AS, <mailto:andreas.solberg@uninett.no>
* Version 0.1. *Last edited July 6th 2011.*

This document is currently just some notes, thinking loud about the possibility of building a federation on top of OpenID Connect. If it turns out to be a good idea, these notes may migrate into a spec.

The reason why I even bother to think about this is that I think it is really pity that Federations and the OpenID community is two split worlds. The users (Service Providers) are the ones that are hurt by this, and I don't like that. If there is only a slight possibility that these worlds could meet, I think it is worth exploring.


**THIS DOCUMENT IS A MESS. It is work in progress**


*Disclaimer: My experiences from the OpenID and OAuth world are limited. Writing this document is my way of getting to know the OpenID Connect idea better.*


Useful references:

* OpenID Connect: <http://openidconnect.com/>
* OAuth 2.0: <http://tools.ietf.org/html/draft-ietf-oauth-v2>


[OpenID Connect Discovery]: http://openidconnect.com/#discovery
[WebFinger]: http://hueniverse.com/2009/08/introducing-webfinger/
[JRD]: http://hueniverse.com/2010/05/jrd-the-other-resource-descriptor/
[JSON Simple Sign]: http://jsonenc.info/jss/1.0/
[host-meta]: http://hueniverse.com/2009/11/host-meta-aka-site-meta-and-well-known-uris/
[Magic Signatures]: http://salmon-protocol.googlecode.com/svn/trunk/draft-panzer-magicsig-01.html



## Terminology

I use the Service Provider and Identity Provider terms from the *federation world* on purpose.



## Metadata

Examples given in the following sections shows metadata for a single entity. Usually metadata is presented in a list.

Metadata is represented in a simple JSON object.

TODO: Consider wrap the metadata in [JRD][].




### Identity Provider Metadata

The following properties may be associated with the Identity Provider:


**Generic information**

`title`
: A good name (in default language) of the Identity Provider. Should be short and descriptive.

`descr`
: A description. Not more than one sentence.

`country`
: Two-letter code in captial letters for country of the login provider. The value `_all_` means that the entity is associated with all countries.

`geo`
: A list of geographic coordinates of the Identity Provider.

`keywords`
: A list of searchable keywords associated with the Identity Provider


**OAuth specific information**

`oauth.endpoints.authorization`
: OAuth Authorization Endpoint

`oauth.endpoints.token`
: OAuth Token Endpoint


**Federation properties**

`federation.endpoints.association`
: The association endpoint, for getting a temporary consumer key/secret. Discussed later.


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
		
		oauth.endpoints.authorization: 'https://foodl.org/oauth/auth',
		oauth.endpoints.token:  'https://foodl.org/oauth/token',
		federation.endpoints.association:  'https://foodl.org/oauth/assoc'
		
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
		]
	}



### Federation Metadata

`title`
: Title of the circle of trust. For easier reckognition in logs and configuration. 

`federation.public_key`
: A list of public keys for the Federation operator. Should be represented using *The Magic Envelope Compact Serialization* [Magic Signatures][].

`federation.supported_algs`
: A list of supported algorithms.

`endpoint.trusted-providers`
: An endpoint listing trusted providers.

`endpoint.trusted-consumers`
: An endpoint listing trusted consumers.


Example:

	{
		title: 'eduGAIN',
		endpoint.trusted-providers: 'http://federation.org/trusted-providers',
		endpoint.trusted-consumers: 'http://federation.org/trusted-consumers',
		federation.public_key: [
			'RSA.mVgY8RN6URBTstndvmUUPb4UZTdwvwmddSKE5z_jvKUEK6yk1u3rrC9yN8k6FilGj9K0eeUPe2hf4Pj-5CmHww.AQAB',
		],
		federation.public_key: [
			'RSA-SHA256', 'RSA-SHA1'
		]
	}


## Federation Trust

Let us introduce a third party asserting a set of trusted Identity Providers and Service Providers (circle of trust).

To establish a federation, there will be created two documents:

* Trusted Service Providers, and
* Trusted Identity Providers

served over plain HTTP on each endpoint.

Each document is a list of Service Provider (or Identity Provider respectivey) metadata objects wrapped in a metadata container, and signed using [JSON Simple Sign][].

The metadata container looks like this:

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


After the document is signed document may look like this:

	{
		"data": "afb98af7b87af687f6876ba87f687a5b76a5..."
		"alg": "RSA-SHA256",
		"sigs": [{
			value: "EvGSD2vi8qYcveHnb-rrlok07qnCXjn8YSeCDDXlbhILSabgvNsPpbe76up8w63i2fWHvLKJzeGLKfyHg8ZomQ",
			keyhash: "4k8ikoyC2Xh+8BiIeQ+ob7Hcd2J7/Vj3uM61dy9iRMI="
		}]
	}

TODO: The example above is not correct; should be updated to reflect [JSON Simple Sign][] if we end up using that.





## Dynamic Association

The consumer sends a POST request to the `federation.endpoints.association` endpoint in the Identity Provider metadata.

The body of the request will be a Dynamic Assocation Request encoded in JSON and signed using JSON Simple Sign, using the key associated with one of the public keys listed in the metadata of the consumer.

Here is an example of the `Dynamic Association Request`:

	{
		type: 'client_associate'
		redirect_uri: 'https%3A%2F%2Ffoodl.org%2Foauth%2Fcallback'
	}

The provider then tries to match the `redirect_uri` from the `oauth.endpoints.redirection` properties found in the trusted consumer metadata.

TODO: Consider add more parameters to the request; in example request for attribute profiles etc, level of assurance (if the IdP in example support 2-factor).

TODO: Discuss with the OpenID Connect folks how this message can be signed in the best way. If the request is made as query string parameters, then HTTP-REDIRECT type signatures can be used. If the message is sent in the body as `application/x-www-form-urlencoded` then how should we sign it? Here I have proposed to send the request body JSON encoded instead, and then use [JSON simple Sign][] for the signature.

If the signature was valid, and the provider trust the consumer, the provider will respond with a new consumer key / secret pair, like this:

	{
		client_id: 'consumer',
		client_secret: 'key',
		expires_in: 3600,
		flows_supported: ['web_server', 'user_agent'],
		user_endpoint_url: '',
	}





## Notes


TODO List:

* Metadata
* Federation trust through a third party
* Dynamic association bootstrapped from the federation trust
* Discovery (or Directory of Providers)
	* Compatability with [WebFinger][]
	* Compatability with [OpenID Connect Discovery][]
	* Consider [host-meta][]
* Automatic exchange of bi-lateral trust relationships within the federation.
* Automated negotiation of attribute release




Important:

* Re-introduce some crypto in OAuth.
* Add federation as a layer on top of OAuth 2.0, without modifying anything on the base layer. This will ensure simple deployment of Consumers that would like to both connect to federations and open providers.
* Keep it simple.
* Central component needs to be super light weight.







