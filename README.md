Branch-Public-API
=================

a public API to tie into for fancy integrations. All endpoints are appended to **https://api.branch.io**

### Creating a deeplinking URL

#### Endpoint

    POST /v1/url
    Content-Type: application/json

#### Parameters

**app_id** _required_
: The id of the originating app

##### Functional

**data** _optional_
: The dictionary to embed with the link. Accessed as session or install parameters from the SDK

**Note** 
You can customize the Facebook OG tags of each URL if you want to dynamically share content by using the following optional keys in the data dictionary:
"$og_app_id"
"$og_title"
"$og_description"
"$og_image_url"

Also, you do custom redirection by inserting the following optional keys in the dictionary:
"$desktop_url"
"$android_url"
"$ios_url"
"$ipad_url"

**type** _optional_
: ADVANCED: Set type to 1, to make the URL a one-time use URL. It won't deep link after 1 successful deep link

##### Tracking

**identity**  _optional_
: The identity used to identify the user. If the link is not tied to an identity, there's no need to specify an identity

**tags** _optional_
: An array of strings, which are custom tags in which to categorize the links by. Recommended syntax: "tags":[t1,t2,t3]

**campaign** _optional_
: the campaign in which the link will be used. eg: "new_product_launch", etc

**feature** _optional_
: the feature in which the link will be used. eg: "invite", "referral", "share", "gift", etc

**channel** _optional_
: the channel in which the link will be shared. eg: "facebook", "text_message"

**stage** _optional_
: A string value that represents the stage of the user in the app. eg: "level1", "logged_in", etc



#### Returns

    {
        'url': 'http://bnc.lt/l/deeplink-randomID'
    }

### Structuring a 'dynamic' deeplink

This should be used for situations where the longer link is alright and you want to create links on the fly without a POST to the API.

#### Endpoint

    GET https://bnc.lt/a/:app_id?AnyOptionalQueryParamsBelow

	Example:
	https://bnc.lt/a/:app_id?data=ExampleBase64EncodedString&has_app=no&channel=facebook&stage=level4&feature=affiliate

#### Parameters

**app_id** _required_
: The id of the originating app

##### Functional

**data**  _optional_
: Default is { }. Base 64 Encoded JSON dictionary of parameters to pass into the app. Default redirects can be overridden with $ios_url, $android_url and $desktop_url. The appearance in social media can be customized with the $og_title, $og_description and $og_image_url keys.

**has_app** _optional_
: Default is 'no'. Possible values are 'yes' or 'no'. If you specify 'yes', we'll try to open up the app immediately instead of sending the clicker to the app store.

**type** _optional_
: Default is 0. Possible values are 0 or 1. A type of 0 means that the link will pass parameters through install any time that it is clicked and followed by an app session. A type of 0 is a security measure, which prevents the link from passing parameters into the app after the first successful deep link

##### Tracking

**tags** _optional_
: An array of strings, which are custom tags in which to categorize the links by. Recommended syntax: ?tags=a&tags=b&tags=c

**campaign** _optional_
: the campaign in which the link will be used. eg: "new_product_launch", etc

**feature** _optional_
: the feature in which the link will be used. eg: "invite", "referral", "share", "gift", etc

**channel** _optional_
: the channel in which the link will be shared. eg: "facebook", "text_message"

**stage** _optional_
: A string value that represents the stage of the user in the app. eg: "level1", "logged_in", etc

### Getting credit count

#### Endpoint

    GET /v1/credits?app_id=[app id]&identity=[identity]

#### Parameters

**app_id** _required_
: The id of the originating app

**identity**  _required_
: The identity used to identify the user.

#### Returns

    {
        'default': 15,
        'other bucket': 4
    }

### Redeeming credits

#### Endpoint

    POST /v1/redeem
    Content-Type: application/json
    
#### Parameters

**app_id** _required_
: The id of the originating app

**user_id** _required_ 
: The dashboard user id. This will be sent to you by the Branch team to give you access to this API

**identity**  _required_
: The identity used to identify the user.

**amount** _required_
: The amount of credit to redeem.

**bucket** _optional_
: The name of the bucket to use. If none is specified, defaults to 'default'

#### Returns

Nothing if successful, or 402 error if not enough credits were available to redeem (this operation is atomic, meaning if two callers try and redeem the same user's credits at the same time, only one will succeed).

### Getting the credit history

#### Endpoint

    GET /v1/credithistory?app_id=[app id]&identity=[identity]

#### Parameters

**app_id** _required_ 
: The id of the originating app

**identity** _required_ 
: The user ID to retrieve credit history for

**bucket** _optional_ 
: The bucket from which to retrieve credit transactions

**begin_after_id** _optional_ 
: The credit transaction id of the last item in the previous retrieval. Retrieval will start from the transaction next to it. If none is specified, retrieval starts from the very beginning in the transaction history, depending on the order.

**length** _optional_ 
: The number of credit transactions to retrieve. If none is specified, up to 100 credit transactions will be retrieved.

**direction** _optional_
: The order of credit transactions to retrieve. If direction is "asc", retrieval is in least recent first order; If direction is "desc", or if none is specified, retrieval is in most recent first order.

#### Returns
    
    [
        {
            "transaction": {
                               "date": "2014-10-14T01:54:40.425Z",
                               "id": "50388077461373184",
                               "bucket": "default",
                               "type": 0,
                               "amount": 5
                           },
            "referrer": "12345678",
            "referree": null
        },
        {
            "transaction": {
                               "date": "2014-10-14T01:55:09.474Z",
                               "id": "50388199301710081",
                               "bucket": "default",
                               "type": 2,
                               "amount": -3
                           },
            "referrer": null,
            "referree": "12345678"
        }
    ]

**referrer**
: The id of the referring user for this credit transaction. Returns null if no referrer is involved. Note this id is the user id in developer's own system that's previously passed to Branch's identify user API call. 

**referree**
: The id of the user who was referred for this credit transaction. Returns null if no referree is involved. Note this id is the user id in developer's own system that's previously passed to Branch's identify user API call. 

**id**
: The id can be used and passed as the "begin_after_id" parameter in the subsequent API call to retrieve the next batch of credit transactions.

### Creating a remote event for funnels

#### Endpoint

    POST /v1/event
    Content-Type: application/json
    
#### Parameters

**app_id** _required_
: The id of the originating app

**user_id** _required_ 
: The dashboard user id. This will be sent to you by the Branch team to give you access to this API

**identity**  _required_
: The identity used to identify the user.

**event** _required_
: The event to associate with this identity

**metadata** _optional_
:  any associated parameters to be stored with the event. 1 layer JSON format

#### Returns

nothing


### Creating a dynamic reward rule

#### Endpoint

    POST /v1/eventresponse
    Content-Type: application/json
    
#### Parameters

**app_id** _required_
: The id of the originating app

**user_id** _required_ 
: The dashboard user id. This will be sent to you by the Branch team to give you access to this API

**calculation_type**  _required_
: This defines whether the rule can reward a user indefinitely, or a single time.

1. _0_ - reward a user continually for the action
1. _1_ - user is only eligible for single reward

**location** _required_
: The user to reward for the action

1. _0_ - the user completing the action receives credit
1. _1_ - the user who referred the user completing the action receives credit

**type** _required_
: the type of event response

1. _"web_hook"_ - register for a web hook callback when the criteria are met
1. _"credit"_ - for referral based rewards, reward the user who caused the referred install
1. _"credit_session"_ - for referral based rewards, reward the user who referred the a new session

**event** _required_
: The event string to trigger the reward, eg "completed_purchase"

**metadata** _required_
: The metadata to define the event response, in JSON format.

For web hooks, use the following keys:

1. _"web_hook_url"_ - the url to call when an event happens

For credits, use the following keys;

1. _"amount"_ - the amount to reward the user
1. _"bucket"_ - the bucket to deposit the amount into

**filter** _optional_
: This is the set of keys and values that must be contained in the event metadata for this reward to be issued, in JSON format.

#### Returns

nothing

### Getting current Branch app config

#### Endpoint

	GET /v1/app/:app_id?user_id=[user id]

#### Parameters

**app_id** _required_ 
: The id of the originating app

**user_id** _required_ 
: The dashboard user id. This will be sent to you by the Branch team to give you access to this API

#### Returns

	{
		app_key: "the app key",
		creation_date : "date app was created",
		
		app_name: "name of the app",
		
		dev_name: "main contact name",
		dev_email: "main contact email",
		dev_phone_number: "main contact phone",
		
		android_url: "url of Android store, or namespace (com.android.myapp)",
		android_uri_scheme: "the Android URI scheme",
		
		ios_url: "url of iOS store, or app id (id512451233)",
		ios_uri_scheme:  "the iOS URI scheme",
		ios_store_country: "the country code of the app, default to US",
		
		web_url: "backup website if URLs are null",
		
		short_url_domain: "white labeled domain for short links",
		
		text_message: "text message to use, {{ link }} will be replaced with short link",
		
		og_app_id: "optional default Open Graph (OG) app id",
		og_title: "optional default OG title",
		og_image_url: "optional default OG image URL",
		og_description: "optional default OG description"
	}


### Creating a new Branch app config

#### Endpoint

    POST /v1/app
    Content-Type: application/json
    
#### Parameters

**user_id** _required_ 
: The dashboard user id. This will be sent to you by the Branch team to give you access to this API

**app_name** _required_ 
: The name of the app

**dev_name** _required_ 
: The main contact developer name

**dev_email** _required_ 
: The main contact developer email

Note: we'll send an invite message to this email upon account creation.

**dev_phone_number** _optional_ 
: The main contact phone number

**android_url** _optional_ 
: url of Android store, or namespace (com.android.myapp)

**android_uri_scheme** _optional_ 
: the Android URI scheme

**ios_url** _optional_ 
: url of iOS store, or app id (id512451233)

**ios_uri_scheme** _optional_ 
: the iOS URI scheme

**ios_store_country** _optional_ 
: the country code of the app, default to US

**web_url** _optional_ 
: backup website if URLs are null

**text_message** _optional_ 
: text message to use for text-me feature, {{ link }} will be replaced with short link

**og_app_id** _optional_ 
: default Open Graph (OG) app id

**og_title** _optional_ 
: default OG title to be used with links

**og_description** _optional_ 
: default OG description to be used with links

**og_image_url** _optional_ 
: default OG image URL to be used with links

#### Returns

	{
		app_key: "the app key",
		creation_date : "date app was created",
		
		app_name: "name of the app",
		
		dev_name: "main contact name",
		dev_email: "main contact email",
		dev_phone_number: "main contact phone",
		
		android_url: "url of Android store, or namespace (com.android.myapp)",
		android_uri_scheme: "the Android URI scheme",
		
		ios_url: "url of iOS store, or app id (id512451233)",
		ios_uri_scheme:  "the iOS URI scheme",
		ios_store_country: "the country code of the app, default to US",
		
		web_url: "backup website if URLs are null",
		
		short_url_domain: "white labeled domain for short links",
		
		text_message: "text message to use, {{ link }} will be replaced with short link",
		
		og_app_id: "optional default Open Graph (OG) app id",
		og_title: "optional default OG title",
		og_image_url: "optional default OG image URL",
		og_description: "optional default OG description"
	}


### Updating a Branch app config

#### Endpoint

    PUT /v1/app/:app_id
    Content-Type: application/json

#### Parameters

**app_id** _required_ 
: The id of the originating app

**user_id** _required_ 
: The dashboard user id. This will be sent to you by the Branch team to give you access to this API

**app_name** _optional_ 
: The name of the app

**dev_name** _optional_ 
: The main contact developer name

**dev_email** _optional_ 
: The main contact developer email

**dev_phone_number** _optional_ 
: The main contact phone number

**android_url** _optional_ 
: url of Android store, or namespace (com.android.myapp)

**android_uri_scheme** _optional_ 
: the Android URI scheme

**ios_url** _optional_ 
: url of iOS store, or app id (id512451233)

**ios_uri_scheme** _optional_ 
: the iOS URI scheme

**ios_store_country** _optional_ 
: the country code of the app, default to US

**web_url** _optional_ 
: backup website if URLs are null

**text_message** _optional_ 
: text message to use for text-me feature, {{ link }} will be replaced with short link

**og_app_id** _optional_ 
: default Open Graph (OG) app id

**og_title** _optional_ 
: default OG title to be used with links

**og_description** _optional_ 
: default OG description to be used with links

**og_image_url** _optional_ 
: default OG image URL to be used with links

#### Returns

	{
		app_key: "the app key",
		creation_date : "date app was created",
		
		app_name: "name of the app",
		
		dev_name: "main contact name",
		dev_email: "main contact email",
		dev_phone_number: "main contact phone",
		
		android_url: "url of Android store, or namespace (com.android.myapp)",
		android_uri_scheme: "the Android URI scheme",
		
		ios_url: "url of iOS store, or app id (id512451233)",
		ios_uri_scheme:  "the iOS URI scheme",
		ios_store_country: "the country code of the app, default to US",
		
		web_url: "backup website if URLs are null",
		
		short_url_domain: "white labeled domain for short links",
		
		text_message: "text message to use, {{ link }} will be replaced with short link",
		
		og_app_id: "optional default Open Graph (OG) app id",
		og_title: "optional default OG title",
		og_image_url: "optional default OG image URL",
		og_description: "optional default OG description"
	}
