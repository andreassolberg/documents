# OAuth 2.0 Metadata

Metadata representing an OAuth 2.0 Provider for a specific client.

issuer
:	lskdjflkjsf

foo
:	sldkjflsdkjf


	{
		"issuer": "https://example.provider.org",
		
		"endpoints": {
			"authorization": "https://example.provider.org/oauth/authorization",
			"token": "https://example.provider.org/oauth/token",
		},
		"supportedScopes": ["openid"],
		
		"client": {
			"redirect_uri" "http://client."	
		}
		
	}





# OpenID Connect Metadata

Currently only a set of examples. 

## Entity Metadata

The metadata is configuration of the entity that needs to be tested by the tool.

### OpenID Connect Consumer

This is the configuration object (metadata) of a Consumer that is about to be tested by the test tool.

	{
		// The identifier of this consumer
		'client_id': 'testConsumer',
		
		// Which user data schema is supported by this consumer.
		'schema': ['openid'],
		
		'redirect_uri': [
			'http://consumer.example.org/callback'
		]
	}


### OpenID Connect Provider

This is the configuration object (metadata) of a Provider that is about to be tested by the test tool.

	{
		// The identifier of this Provider
		'issuer': 'provider',
		
		// Client credentials of a client that is trusted by the provider.
		// This needs to be configured by the provider together with the
		// 'redirect_uri' of the test utility. 
		'client_auth': {
			
			/* How will the client authenticate towards the Token endpoint.
			 * basic: HTTP Basic authentication
			 * none: No authentication
			 */
			'authorization': 'basic,
			'client_id': 'testConsumer',
			'password': 'secret'
		},
		
		/*
		 * Definition of user interaction on the Provider.
		 * A set of rules. Which rule to execute to decided based upon the 'identify'
		 * parameter
		 */
		'interactions': [
			
			{
				'identify': {
					// The URL Match should not include query string parameters when matching..
					'url': 'http://provider.example.org/login'
				},
				'type': 'form',
				'form': {
					// All form parameters that are not mentioned here are bypassed with the default value.
					// Form parameters that are set to a string is overrided,
					// form parameters that are set to false are ignored.
					'username': 'andreas',
					'password': 'secret',
					'lang': false
				}
			},
			
			{
				'identify': {
					// Identify this rule by the 'id' attribute of the <form> element on the page.
					'form-id': 'consent-new-provider'
				},
				'type': 'form',
				'form': {
					'accept': 'checked',
					'submit': 'OK'
				}
			}
			
		],
		
		// A set of flags for features that the Provider supports
		'supports': {
			'implicit': true,
			'dynamicregistration': false,
			'operationswithoutcookie': false
		}
	
		// Which user data schema is supported by this provider.
		'schema': ['openid'],
		'endpoints': {
			authorization: 'http://provider.example.org/authorization',
			token: 'http://provider.example.org/token',
			checksession: 'http://provider.example.org/session/check',
			refreshsession: 'http://provider.example.org/session/refresh',
			endsession: 'http://provider.example.org/session/end'
		}
	}







