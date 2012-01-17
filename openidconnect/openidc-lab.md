# OpenID Connect Lab Interfaces

Currently only a set of examples. 


## Test flow definitions

The test flow definitions are static for each version of the test tool. The test tool may store this as a separate defintion file, or return it somehow.


	[
		{
			id: 'basic-code-idtoken',
			name: 'Basic Code flow with ID Token',
			descr: 'Very basic test of a Provider using the authorization code flow. The test tool acting as a consumer is very relaxed and tries to obtain an ID Token.'
		},
		{
			id: 'basic-code',
			name: 'Basic Code flow with ID Token and User data',
			descr: 'Very basic test of a Provider using the authorization code flow, but in addition to retrieve an ID Token, this test flow also tried to obtain user data.',
			depends: ['basic-code-idtoken']
		},
		{
			id: 'extended-code',
			name: 'Extended Code flow',
			descr: 'Extended test of a Provider using the authorization code flow. Both validating the ID Token and testing the User data service.',
			depends: ['basic-code-idtoken', 'basic-code']
		},
		{
			id: 'basic-implicit',
			name: 'Basic Implicit Grant',
			descr: 'Very basic test of the implicit grant flow.'
		},
		{
			id: 'external-authorization-request-file',
			name: 'External Request File',
			descr: 'Tries to send a request referring to a external request file object'
		},
		{
			id: 'unknown-scope',
			name: 'Unknown scope in authorization request',
			descr: 'A bogus unknown scope is included in the authorization request'
		},
		{
			id: 'display-none',
			name: 'No user interaction with display: none',
			descr: 'First, enable a Single Sign-On session, then send a new authorization request with display: none and verify that things work. Second, without enabling SSO, send a display: none, and verify that the correct status code is returned, and no user interaction is provided.'
		},
		{
			id: 'no-coookie',
			name: 'Test standard Code flow with cookie support turned off.',
			// Will not run this test unless all test flow dependencies are successfully run.
			depends: ['basic-code-idtoken'],
			// Requires some features in metadata to run this test flow.
			requires: ['operationswithoutcookie']
		}
	]





## Test flow output


Definition of status values:

* **0**: Failure
* **1**: Success
* **2**: Information
* **9**: Test flow interrupted. Could not continue to run test because of this problem.

Signficiance levels of test status:

* **3**: Signficant fatal error. Operation unlikely to work with any client.
* **2**: Spec violation; Spec violation and/or security issue.
* **1**: Warning; less significant, unclear in the spec.
* **0**: Information: There is no right and wrong in this test, it just informs about behavior of the entity (that might be useful).

The significance levels may be overridden by configuration of various deployment profiles.


	{
		id: 'extended-code',
		
		// The aggregated status of all the tests in this test flow.
		// The aggregated status code may only be 0 or 1.
		status: 1,
	
		tests: [
		
			{
				id: 'connectivity',
				name: 'Connectivity with the authorization endpoint',
				status: 1,
				significance: 3
			},
		
			{
				id: 'userdata-content-type',
				name: 'User Data Endpoint returned correct MIME type in the Content-Type header: application/json',
				status: 1,
				significance: 2,
			
				// A reference to the specification may be included. The test UI will be able to include a clickable link to the 
				// relevant text in the specification.
				reference: {
					document: 'oauth',
					version: 'draft-08',
					section: '3.2.1.2.3'
				}
			},
		
			{
				id: 'obtained-refresh-token',
				name: 'Client did not receive a refresh token in the respone from the token endpoint.',
				status: 0,
				significance: 0
			},
		
			{
				id: 'access-token-duration',
				name: 'The duration of the access token was 3600 seconds',
				status: 2
				// Signficance is optional, and does not make sense when status is set to 2.
			},
		
		
		]
	}







