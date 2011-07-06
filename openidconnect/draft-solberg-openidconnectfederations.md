# OpenID Connect Federations

* Identifyer: draft-solberg-openidconnectfederations
* Editor: Andreas Åkre Solberg, UNINETT AS, <mailto:andreas.solberg@uninett.no>
* Version 0.1. *Last edited July 6th 2011.*

This document is currently just some notes, thinking loud about the possibility of building a federation on top of OpenID Connect. If it turns out to be a good idea, these notes may migrate into a spec.

The reason why I even bother to think about this is that I think it is really pity that Federations and the OpenID community is two split worlds. The users (Service Providers) are the ones that are hurt by this, and I don't like that. If there is only a slight possibility that these worlds could meet, I think it is worth exploring.





*Disclaimer: My experiences from the OpenID and OAuth world are limited. Writing this document is my way of getting to know the OpenID Connect idea better.*


Important:

* Re-introduce some crypto in OAuth.
* Add federation as a layer on top of OAuth 2.0, without modifying anything on the base layer. This will ensure simple deployment of Consumers that would like to both connect to federations and open providers.
* Keep it simple.
* Central component needs to be super light weight.



Useful references:

* OpenID Connect: <http://openidconnect.com/>
* OAuth 2.0: <http://tools.ietf.org/html/draft-ietf-oauth-v2>


TODO List:

* Metadata
	* SP Metadata
	* Provider Metadata
	* Federation Metadata
* Federation trust through a third party
* Dynamic association bootstrapped from the federation trust
* Discovery (or Directory of Providers)
	* Compatability with [WebFinger][]
	* Compatability with [OpenID Connect Discovery][]
	* Consider [host-meta][]
* Automatic exchange of bi-lateral trust relationships within the federation.
* Automated negotiation of attribute release


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

<!-- 

**Trust configuration**

The metadata data format may be used in a number of places. When the metadata is used for local configuration, the document may include a client key / secret for simplicity.

`oauth.consumer_key`
: OAuth Authorization Endpoint

`oauth.consumer_secret`
: OAuth Token Endpoint

**These parameters may be useful for the generic metadata format, but will not be used with OpenID Connect Federations**

-->


**Additional Federation stuff**



Example metadata:

	{
		title: 'Norwegian University of Science and Technology',
		descr: 'The Norwegian University of Science and Technology (NTNU) is Norway’s primary institution for educating the nation's future engineers and scientists.',
		keywords: ['NTNU', 'NUST', 'Trondheim'],
		country: 'NO',
		geo: [{lat: 63.4171494, lon: 10.40447}],
		
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


Example metadata:

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

`signing.public_key`
: public signing key

`endpoint.trusted-providers`
: 

`endpoint.trusted-consumers`
: A list of trusted consumers


## Federation Trust and Dynamic Association

Let us introduce a third party asserting a set of trusted Identity Providers and Service Providers (circle of trust).

To establish a federation, there will be created two documents:

* Trusted Service Providers, and
* Trusted Identity Providers

served over plain HTTP on each endpoint. These documents could in example be served on 

* `http://federation.org/trusted-providers`
* `http://federation.org/trusted-consumers`

Each document is represented as a JSON array with metadata, signed using [JSON Simple Sign][].


### Trusted (Identity) Providers


	"alg": "RSA-SHA256",
	"sigs": [
	  {
	  "value": "EvGSD2vi8qYcveHnb-rrlok07qnCXjn8YSeCDDXlbhILSabgvNsPpbe76
	  up8w63i2fWHvLKJzeGLKfyHg8ZomQ",
	  "keyhash": "4k8ikoyC2Xh+8BiIeQ+ob7Hcd2J7/Vj3uM61dy9iRMI="
	  }
	]


### Trusted Consumers (Service Providers)







## Dynamic Association

The 




federation_client_key
federation_client_secret





