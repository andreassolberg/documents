# OpenID Connect Dynamic Client Registration and Metadata Refresh Service

Author: Andreas Åkre Solberg, andreas.solberg@uninett.no http://rnd.feide.no

*DICLAIMER: Only brainstorming at this point. Not even close to a draft.*

This specification defines a way to automate the registration of clients at the Provider, and a way for the provider and consumer to automatically refresh the metadata about the other party.

## Metadata Refresh Service

Both the Provider and Consumer may implement a *Metadata Refresh Service* which responds to a GET request with the JSON metadata.

OpenID Connect Metadata is an independent spec, currently described here: <https://gist.github.com/1163089>

The Metadata Consumer (may be the OpenID Connect Provider or Consumer, or any other parties) performs a GET request to the `metadata/endpoint` of the Metadata Provider. The request includes the following parameters:

* `id` - The Provider identifier or the Consumer Identifier. OPTIONAL.

The request should indicate whether the JWT or the JSON format is needed in the Accept-content-type header.

The provider responds with a metadata JSON document (if applicable encoded as a JWT) with the corresponding Content-type header set.

Example:

	GET /provider/metadata?id=https%3A%2F%2Fprovider.example.org HTTP/1.1
	Host: provider.example.org
	Accept: application/json
	
	{'client_id': 'https://consumer.example.org', 'redirect_url': 'https://consumer.example.org/callback'}


A Metadata JWT MUST be signed using a shared secret known to both parties or a public key also present in the metadata. The Metadata Provider may use the `id` parameter in the request to lookup the appropriate shared secret.

The Metadata Consumer SHOULD never trust a metadata JWT that is expired (or otherwise not valid). The Metadata Consumer should refresh the metadata JWT using the same service, before it expires. This allows the Metadata Provider to update metadata, and distribute the new metadata automatically.

The Metadata Consumer MAY include a new metadata_url in the metadata JWT, the metadata consumer MUST then perform subsequent request for metadata to the new endpoint.



## Step 1: The client learns about the Provider

The client is fed with the `metadata/endpoint` of the Provider, and possibly also the public keys or a shared secret with the provider.

The client uses the *Metadata Refresh Service* Endpoint to obtain metadata about the provider.

*This step may be replaced with a more manual procedure if appropriate for better verification.*




## Step 2: Client associates the client ID and credentials


*This step may be replaced with a more manual procedure if appropriate for better verification.*


The client sends an associate request to the `client_registration/endpoint` of the provider (if found in metadata obtainer earlier).

The associate request is a HTTPS GET request with the following parameters:

* `action` - The value MUST be `associate` - REQUIRED
* `client_id` - The requested client_id of the  - REQUIRED
* `alg` - Select one of the supported algorithm for signing messages, selected from the `crypto/supported_algs` list in the provider metadata. - REQUIRED

Depending on the `alg` parameter value, additional parameters may be provided.

If the association was succcessful, the Provider returns a response with the HTTP status code of 200. The provider should then return a response object:

	{
		'connect_code': '238476238746238746238746283746'
	}

with the content type of `application/json`

Example request:

	GET /provider/register?action=associate&client_id=https%3A%2F%2Fconsumer.example.org&alg=HS256 HTTP/1.1
	Host: provider.example.org
	Accept: application/json

And response:

	HTTP 200 OK
	Content-Type: application/json
	
	{'connect_code': '238476238746238746238746283746', 'secret': '2189763287468A2934792374BF23847D2'}


### Association with shared secret.

If the `alg` parameter is set to one of `HS256`, `HS512`, the response MUST include an additional parameter `secret` which will be the shared secret used for subsequent communication.

The Provider MUST never return the secret more than one time for a client_id. The `client_id` should then be blocked for a certain time (1-24 hours).

### Association with public keys

If the alg is set to one of `rsa...`, the association request MUST include the following additional parameters:

* public key... somehow..


## Step 3: The Client connects to the provider, by providing its metadata

The client sends a `connect` request to the provider, by redirecting the user that registers the service to the registration endpoint with the following parameters:

* `action` - The value MUST be `connect` - REQUIRED
* `client_id` - The requested client_id of the  - REQUIRED
* `connect_code` - The code (secret) obtain in the *association* phase. - REQUIRED
* `metadata_url` - Select one of the supported algorithm for signing messages, selected from the `crypto/supported_algs` list in the provider metadata. - REQUIRED
* `redirect_url` - Where to send the user back after connecting (if done via a front-channel redirect)

In some situations the provider may accept this request to be sent backchannel without a end user registering the service. In that case the `redirect_url` should not be provided, and the Provider returns a response with the HTTP status code of 200. The provider should then return a response object:

	{
		'client_id': 'https://consumer.example.org'
	}

with the content type of `application/json`.


If the `redirect_url` was present, the Provider MAY authenticate, verify and ask for confirmation of the connect between the service and the provier.

The Provider returns a successfull response, by redirecting the user back to the `redirect_url` with the following parameters:

* `iss`: The Provider ID
* `connect`: The value is set to `ok`.

Sending a `connect` request MAY be done to push the Provider to refresh its metadata (back-channel). But if the Provider already has connected to the Consumer, the Provider MUST never allow the metadata URL to be replaced by this request.


An example front-channel connect request:

	GET /provider/register?
		action=connect&
		client_id=https%3A%2F%2Fconsumer.example.org&
		connect_code=238476238746238746238746283746&
		metadata_url=https%3A%2F%2Fconsumer.example.org%2Fmetadata&
		redirect_url&https%3A%2F%2Fconsumer.example.org%2Fcallback
	Host: provider.example.org
	Accept: application/json

And response:

	HTTP 302 Redirect
	Location: https://consumer.example.org/callback?connect=ok&iss=https%3A%2F%2Fprovider.example.org


## Step 4: Provider obtains Consumer metadata

The Provider now contacts the `metadata_url` in the connect request, and verifies the signature against the trust information exchanged in the `associate` request.

This step usually goes after the Step 3 request is sent and before the step 3 response is sent. TBD: restructure.





