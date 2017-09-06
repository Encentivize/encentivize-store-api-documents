#Voucher Store#

This api interface allows you to interact with the encentivize system via [REST](https://en.wikipedia.org/wiki/Representational_state_transfer).

- Testing can be done in our QA environment.
- All routes require your credentials supplied via basic authentication. Contact your account manager for QA & live credentials.

## Authentication ##
The api uses the [basic authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) scheme. So for example:

	Username - test
	password - test12345 
	header - Authorization: Basic dGVzdDp0ZXN0MTIzNDU= 

This authentication header must be provided on every request to the api. If you do not you will get a 401 Unauthorised response with "Invalid username or password" in the body of the response.

## Qa & Live Details ##
- Contact your account manager to get the url and credentials.

## Paging ##
The list of partners and products both make use of paging through the use of two query string parameters :

	?pageSize=10&pageNumber=1
By manipulating the numerical values above you can request different pages and items per page. The response will contain the following paging info:

	{
	    "data": [
	        ... // The actual data returned in this response
	    ],
	    "firstItem": 1, // The index of the first item returned in data within the complete, unpaged data structure 
	    "hasNextPage": true, // Does the server have a next page of data
	    "hasPreviousPage": false, // Does the server have a previous page of data
	    "lastItem": 10, // The inde of the last item returned in data within the complete, unpaged data structure
	    "pageNumber": 1, // Current page number
	    "pageSize": 10, // Current page size
	    "totalItems": 26, // The total number of items
	    "totalPages": 3 // The total number of pages
	}

## Example Workflow ##
These examples will refer to api methods below by number. e.g. (1) refers to the heading *1. Get partners* below.

1. Get a list of partners from our api using (1). These partners act as containers or categories, grouping vouchers from the same suppliers together.
2. Once the user has selected a partner, you can find the products provided by that partner using (2).
3. Once the user has selected a product they would like to redeem, you can request it using (3).
4. (Optional) the encentivize system can be configured to call your system to notify you of successful and failed voucher redemption's. See below for details.

## Api Functions ##

### 1. Get partners ###
- Route:  **/partners**
- HTTP Method: **GET**
- Url parameters: **none**

returns : 

	{
		"data": [{
		    "partnerId": 4, // The unique identifier for the partner
		    "displayName": "Exclusive Books", // The display name for the partner
		    "description": "Exclusive Books", // The long description for the partner
		    "logoUrl": "http://www.vectortemplates.com/raster/batman-logo-big.gif" // The url for the partner's logo
		}],
		... paging info...
	}

### 2. Get Products For Partner ###
- Route:  **/partners/:partnerId/products**
- HTTP Method: **GET**
- Url parameters: **partnerId** [required]

returns : 

	{
		"data": [{
			"productId" : 45, // The unique identifier for the product 
			"partnerId" : 3, // The unique identifier for the partner that supplies the product
			"name" : "Vida Cafe R30", // The display name of the product
			"description" : "R30 Vida Voucher", // The long description of the product. This field supports HTML.
			"salesPriceIncVat" : 30, // The sales price for this product, including vat
			"salesPriceExcVat" : 30, // The sales price for this product, excluding vat
			"salesVat" : 0, // The vat portion of the Sales Price
			"validFromDateUtc" : "2014-08-06T11:28:41.257Z", // The ISO date string indicating when this product was first valid, products that are not yet valid will not be returned by the api.
			"validToDateUtc" : "2034-08-06T11:28:41.257Z", // The ISO date string indicating when this product will be available until. Products that are no longer valid will not be returned by the api, this date is subject to change without notice.
			"termsAndConditions" : "- Partial redemption is not allowed i.e. no change is given for a voucher.\n- Voucher codes will be delivered via email. \n- There may be a 2 day lead time on low stock.", // This field lists the terms and conditions of the voucher
			"usageInstructions" : null, // This field contains any special requirements that need to be followed in order to use the provided voucher
			"slaTime" : 2, // This is the maximum time in days that the voucher will take to be fulfilled 
			"slaText" : "Voucher codes will be delivered via email", // This is the description of the SLA
			"imageUrl" : null, // This is the url to the voucher.
			"imageCaption" : null, // This is the description of the image, often used for the alt text in html
			"currentStockBalance": 46 //The current stock level for this product
			"units" : 1 //the number of stock units that are required when redeeming one of these rewards. e.g. if the units is set to 5 and the currentStockBalance is 10, then you can redeem two of these rewards.
			"maxVariableValue":null //the maximum value that this voucher can be issued for. Only applies to variable value vouchers
			"minVariableValue":null //the minimum value that this voucher can be issued for. Only applies to variable value vouchers
		}],
		... paging info...
	}

### 3. Redeem Product ###
- Route:  **/products/:productId/redeem**
- HTTP Method: **POST**
- Url parameters: **productId** [required]

POST parameters :

	{
	  "recipient": { // This object contains the information about the recipient who will be recieving the voucher 
	    "reference":"uniqueMemberReference", // This is the unique identifier for the recipient so that the request can be tied back to your system
	    "displayName": "displayName that will go into comms", // The display name for the member, this will often be used in communications
	    "primaryMobile" : "082 555 5555", // The primary mobile phone number for the member, this is where sms's with the voucher codes will be sent
	    "primaryEmail" : "testUser@company.co.za", // The primary email address for the member, this is where emails with the voucher codes will be sent
	    "someCustomField1" : true, // You can also include any extra properties about the member that relate to the transaction in the standard JSON format
	    "moreCustomFields" : "asd"
	  },
	
	  "variableValue":5  //The value of the issued voucher. Only applicable to variable value vouchers (see 3.1)
	}

returns : 
	
	{
	    "_id": "55e95c04709826e815333b32", // the unique identifier for the order
	    "productId": 10, // the product id requested
	    "programName": "testProgram", // the program name
	    "quantity": 1, //the number of items in the order
	    "partnerId": 3, // the partner id for the order
	    "status": "pending" // the current status of the order
	}

- If successful will return 2xx with the response object above.
- If any of the supplied parameters are wrong it will return 4xx with a description of what's wrong.
- If there is no stock available will return 4xx with the no stock description.
- If there is insufficient funds in your account, then it will return 4xx Bad Request with the insufficient funds description.
- If an error occures when processing the request the api will return a 5xx Server Error.

####3.1 <a name="variable-value">Variable value vouchers</a>
Some vouchers do not have a fixed value, which means you can specify the value that you want the issued voucher to be worth. For example, when redeeming airtime you can specify how much airtime you want. To specify the desired value you must provide the `variableValue` property in the body. The minimum and maximum values for a voucher are restricted by the `maxVariableValue` and `maxVariableValue` on a product (`/products/:productId`). 


### 4. Get Account ###
- Route:  **/account**
- HTTP Method: **GET**
- Url parameters: None

returns : 

	{
		"balance" : 100.50 //Your current account balance,
	    "currency": "ZAR" //The currency that the balance and products are priced in. 
	}
	
## Third party callback (Optional) ##
- A callback url may be specified for both successful and failed voucher requests.
- You can use the same url for both types or provide a different url for each. 
- Both responses will be sent as a POST request containing JSON data with the content-type header set to 'application/json;charset=UTF-8'
- Authentication is not supported.
- Any data returned in the response will be persisted to our database for auditing purposes.
  
### Success object ###
    {
        id: "55e95c04709826e815333b32", // the unique identifier for the order
        programName: "testProgram", // the program name
        recipientReference: "66", // the identifier for the member who received the voucher 
        productId: 10 // the identifier for the product requested
        status : 'Success', // the status of the order, will always be 'Success'
        vouchers: [  //contains a list of the vouchers generated, currently limited to one per request though this will change in future
            {
            	"VoucherCode" : "test-123", // the code the member is to present to the cashier or type in on the website, etc
            	"Value" : 0.0100000000000000, // The rand value of the voucher, currently only supports ZAR but may change in future
            	"VoucherUrl" : "", // The url associated with the voucher, not currently used
            	"Name" : "Test Snack R150", // The name of the voucher
            	"Description" : "Test voucher to the value of R150", // a brief description of the voucher
            	"ValidityDescription" : null, // A description related to the terms under which the voucher is valid
            	"VoucherType" : "", // reserved for future use,
            	"VoucherIssuer" : "", // reserved for future use
            	"ContactInfo" : "",// reserved for future use
            	"VoucherHtml" : "", // reserved for future use
            	"DateIssued" : null, // The ISO date string indicating when this voucher was issued, can be null
            	"ExpiryDate" : null //The ISO date string indicating when the voucher code expires, can be null
            }
        ]
    }

### Failure object ###
    {
        id: "55e95c04709826e815333b32", // the unique identifier for the order
        programName: "testProgram", // the program name
        recipientReference: "66", // the identifier for the member who received the voucher 
        productId: 10 // the identifier for the product requested
        status : 'Failure', // the status of the order, will always be 'Failure'
        errors : [ //contains the list of errors that caused the request to fail, currently only ever a string
            "reason 1",
            "reason 2"
        ]
    }
    
## Working Example ##
If you have Google Chrome installed you can add a plugin called postman and import the example file included with this readme. From there you can call the methods and see examples of the outputs on qa.
Alternatively you can use the [cUrl](http://curl.haxx.se/) scripts provided with this readme to call the endpoints.