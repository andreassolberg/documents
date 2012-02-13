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


Test flow:

  * (1) Succeeded
  * (2) Warning (less significant errors)
  * (3) Error
  * (4) Critical
  * (5) User interaction needed
  	* With html body + url

Test item:

  * (0) Informational
  * (1) Succeed
  * (2) Warning (less significant errors)
  * (3) Error
  * (4) Critical


A test flow looks like this:

	{
		id: 'extended-code',
		// The aggregated status of all the tests in this test flow.
		status: 1,
		tests: [...] 
	}

Test flow properties:

* id: REQUIRED. Unique
* status: REQUIRED: 
* tests: An array with test items.


Test item properties:

* id: REQUIRED.
* name: REQUIRED.
* descr: OPTIONAL.
* status: REQUIRED.


A test looks like this:

	{
		id: 'connectivity',
		name: 'Connectivity with the authorization endpoint',
		status: 1
	},

A more complex test item:

	{
		id: 'userdata-content-type',
		name: 'User Data Endpoint returned correct MIME type in the Content-Type header: application/json',
		descr: "...",
		status: 1,
	
		// A reference to the specification may be included. The test UI will be able to include a clickable link to the 
		// relevant text in the specification.
		reference: {
			document: 'oauth',
			version: 'draft-08',
			section: '3.2.1.2.3'
		}
	},
		







