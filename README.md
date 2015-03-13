Branch-Public-API
=================

A public API to tie into for fancy integrations. All endpoints are appended to **https://api.branch.io**

### Creating a Deep Linking URL

For more details on how to create links, see the [Branch link creation guide](https://github.com/BranchMetrics/Branch-Integration-Guides/blob/master/url-creation-guide.md)

#### Endpoint

    POST /v1/url
    Content-Type: application/json

#### Parameters

**app_id** _required_
: The id of the originating app.

##### Functional

**data** _optional_
: The dictionary to embed with the link. Accessed as session or install parameters from the SDK.

**Note**
You can customize the Facebook OG tags of each URL if you want to dynamically share content by using the following optional keys in the data dictionary. Please use this [Facebook tool](https://developers.facebook.com/tools/debug/og/object) to debug your OG tags!

| Key | Value
| --- | ---
| "$og_title" | The title you'd like to appear for the link in social media
| "$og_description" | The description you'd like to appear for the link in social media
| "$og_image_url" | The URL for the image you'd like to appear for the link in social media
| "$og_video" | The URL for the video 
| "$og_url" | The URL you'd like to appear
| "$og_app_id" | Your OG app ID. Optional and rarely used.

Also, you can set custom redirection by inserting the following optional keys in the dictionary:

| Key | Value
| --- | ---
| "$desktop_url" | Where to send the user on a desktop or laptop. By default it is the Branch-hosted text-me service
| "$android_url" | The replacement URL for the Play Store to send the user if they don't have the app. Currently, Chrome does not support this override. _Only necessary if you want a mobile web splash_
| "$ios_url" | The replacement URL for the App Store to send the user if they don't have the app. _Only necessary if you want a mobile web splash_
| "$ipad_url" | Same as above but for iPad Store
| "$fire_url" | Same as above but for Amazon Fire Store
| "$blackberry_url" | Same as above but for Blackberry Store
| "$windows_phone_url" | Same as above but for Windows Store
| "$after_click_url" | When a user returns to the browser after going to the app, take them to this URL. _iOS only; Android coming soon_

You have the ability to control the direct deep linking of each link as well:

| Key | Value
| --- | ---
| "$deeplink_path" | The value of the deep link path that you'd like us to append to your URI. For example, you could specify "$deeplink_path": "radio/station/456" and we'll open the app with the URI "yourapp://radio/station/456?link_click_id=branch-identifier". This is primarily for supporting legacy deep linking infrastructure. 
| "$always_deeplink" | true or false. (default is not to deep link first) This key can be specified to have our linking service force try to open the app, even if we're not sure the user has the app installed. If the app is not installed, we fall back to the respective app store or $platform_url key. By default, we only open the app if we've seen a user initiate a session in your app from a Branch link (has been cookied and deep linked by Branch).

**alias** _optional_ (max 128 characters)
: Instead of our standard encoded short url, you can specify the alias of the link bnc.lt/alexaustin. Aliases are enforced to be unique per domain (bnc.lt, yourapp.com, etc). Be careful, link aliases are _unique_, immutable objects that cannot be deleted.

NOTE: If you POST to the this endpoint with the same alias, and a matching set of other POST parameters to an existing aliased link, the original will be returned to you. If it clashes and you don't specify a match, will return a HTTP 409 error.

**type** _optional_
: ADVANCED: Set type to 1, to make the URL a one-time use URL. It won't deep link after 1 successful deep link.

**duration** _optional_
: ADVANCED: In seconds. Only set this key if you want to override the match duration for deep link matching. This is the time that Branch allows a click to remain outstanding and be eligible to be matched with a new app session. This is default set to 7200 (2 hours)

##### Tracking

**identity**  _optional_ (max 127 characters)
: The identity used to identify the user. If the link is not tied to an identity, there's no need to specify an identity.

**tags** _optional_ (each max 64 characters)
: An array of strings, which are custom tags in which to categorize the links by. Recommended syntax: "tags":[t1,t2,t3].

**campaign** _optional_ (max 128 characters)
: The campaign in which the link will be used. eg: "new_product_launch", etc.

**feature** _optional_ (max 128 characters)
: The feature in which the link will be used. eg: "invite", "referral", "share", "gift", etc.

**channel** _optional_ (max 128 characters)
: The channel in which the link will be shared. eg: "facebook", "text_message".

**stage** _optional_ (max 128 characters)
: A string value that represents the stage of the user in the app. eg: "level1", "logged_in", etc.



#### Returns

    {
        'url': 'http://bnc.lt/l/deeplink-randomID'
    }

### Bulk creating Deep Linking URLs

For more details on how to create links, see the [Branch link creation guide](https://github.com/BranchMetrics/Branch-Integration-Guides/blob/master/url-creation-guide.md)

#### Endpoint

    POST /v1/url/bulk/:app_id
    Content-Type: application/json

#### Parameters

A json array of pramameters from [Creating a Deep Linking URL](https://github.com/BranchMetrics/Branch-Public-API/blob/master/README.md#creating-a-deep-linking-url)

    [
	{ 'channel': 'branch' },
	{ 'channel': "fb", 'data': '{ "$og_title": "deep linking" }' }
    ]

#### Returns

An array of deep linking urls and/or errors in case invalid params.

    [
     { 'url': 'http://bnc.lt/l/deeplink-randomID' },
     { 'error': 'error message' }  // in case of error
    ]
    
### Structuring a 'dynamic' Deeplink

This should be used for situations where the longer link is okay and you want to create links quickly without a POST to the API.

#### Endpoint

    GET https://bnc.lt/a/<app_id>?AnyOptionalQueryParamsBelow

  Example:
  https://bnc.lt/a/123456789?data=ExampleBase64EncodedString&has_app=no&channel=facebook&stage=level4&feature=affiliate

#### Parameters

**app_id** _required_
: The id of the originating app

##### Functional

**data**  _optional_
: Default is { }. Base 64 Encoded JSON dictionary of parameters to pass into the app. Default redirects can be overridden with $ios_url, $android_url and $desktop_url. The appearance in social media can be customized with the $og_title, $og_description and $og_image_url keys.

**has_app** _optional_
: Default is 'no'. Possible values are 'yes' or 'no'. If you specify 'yes', we'll try to open up the app immediately instead of sending the clicker to the app store.

**type** _optional_
: Default is 0. Possible values are 0 or 1. A type of 0 means that the link will pass parameters through install any time that it is clicked and followed by an app session. A type of 1 is a security measure, which prevents the link from passing parameters into the app after the first successful deep link.

##### Tracking

**tags** _optional_ (each max 64 characters)
: An array of strings, which are custom tags in which to categorize the links by. Recommended syntax: ?tags=a&tags=b&tags=c

**campaign** _optional_ (max 128 characters)
: The campaign in which the link will be used. eg: "new_product_launch", etc.

**feature** _optional_ (max 128 characters)
: The feature in which the link will be used. eg: "invite", "referral", "share", "gift", etc.

**channel** _optional_ (max 128 characters)
: The channel in which the link will be shared. eg: "facebook", "text_message".

**stage** _optional_ (max 128 characters)
: A string value that represents the stage of the user in the app. eg: "level1", "logged_in", etc.

### Getting Credit Count

#### Endpoint

    GET /v1/credits?app_id=[app id]&identity=[identity]

#### Parameters

**app_id** _required_
: The id of the originating app.

**identity**  _required_ (max 127 characters)
: The identity used to identify the user.

#### Returns

    {
        'default': 15,
        'other bucket': 4
    }

### Adding Credits

#### Endpoint

    POST /v1/credits
    Content-Type: application/json

#### Parameters

**app_id** _required_
: The id of the originating app.

**user_id** _required_
: The dashboard user id. This will be sent to you by the Branch team to give you access to this API.

**identity**  _required_ (max 127 characters)
: The identity used to identify the user.

**amount** _required_
: The amount of credit to award.

**bucket** _optional_ (max 63 characters)
: The name of the bucket to use. If none is specified, defaults to 'default'.

#### Returns

`{ success: true }`

### Redeeming Credits

#### Endpoint

    POST /v1/redeem
    Content-Type: application/json

#### Parameters

**app_id** _required_
: The id of the originating app.

**user_id** _required_
: The dashboard user id. This will be sent to you by the Branch team to give you access to this API.

**identity**  _required_ (max 127 characters)
: The identity used to identify the user.

**amount** _required_
: The amount of credit to redeem.

**bucket** _optional_ (max 63 characters)
: The name of the bucket to use. If none is specified, defaults to 'default'.

#### Returns

Nothing if successful, or 402 error if not enough credits were available to redeem (this operation is [atomic](http://stackoverflow.com/questions/15054086/what-does-atomic-mean-in-programming), meaning if two callers try and redeem the same user's credits at the same time, only one will succeed).

### Reconciling Credits

If fraud is detected, e.g. users tricking the system to get more credits by referring him/herself, call this API to reconcile the amount from the user's credit balance. If the reconciliation amount is greater than the user's credit balance, the difference amount will be used and the credit balanced will be set to zero.

#### Endpoint

    POST /v1/reconcile
    Content-Type: application/json

#### Parameters

**app_id** _required_
: The id of the originating app.

**user_id** _required_
: The dashboard user id. This will be sent to you by the Branch team to give you access to this API.

**identity**  _required_ (max 127 characters)
: The identity used to identify the user.

**amount** _required_
: The amount of credit to redeem.

**bucket** _optional_ (max 63 characters)
: The name of the bucket to use. If none is specified, defaults to 'default'.

#### Returns

The credit transaction JSON object for the reconciliation

    {
        "app_id": "55696551485375420",
        "identity_id": "55705482546906094",
        "date": "2015-02-13T02:27:12.477Z",
        "id": "94607584564086264",
        "bucket": "default",
        "type": 4,
        "amount": -5
    }

### Getting the Credit History

#### Endpoint

    GET /v1/credithistory?app_id=[app id]&identity=[identity]

#### Parameters

**app_id** _required_
: The id of the originating app.

**identity** _required_ (max 127 characters)
: The user ID to for which to retrieve credit history.

**bucket** _optional_ (max 63 characters)
: The bucket from which to retrieve credit transactions.

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

**type**
: This is the type of credit transaction.

1. _0_ - A reward that was added automatically by the user completing an action or referral.
1. _1_ - A reward that was added manually.
2. _2_ - A redemption of credits that occurred through our API or SDKs.
3. _3_ - This is a very unique case where we will subtract credits .automatically when we detect fraud.

### Creating a Remote Event for Funnels

#### Endpoint

    POST /v1/event
    Content-Type: application/json

#### Parameters

**app_id** _required_
: The id of the originating app.

**user_id** _required_
: The dashboard user id. This will be sent to you by the Branch team to give you access to this API.

**identity**  _required_ (max 127 characters)
: The identity used to identify the user.

**event** _required_ (max 63 characters)
: The event to associate with this identity.

**metadata** _optional_
:  Any associated parameters to be stored with the event. 1 layer JSON format. (max 255 characters for both keys and values)

#### Returns

nothing


### Creating a Dynamic Reward Rule

#### Endpoint

    POST /v1/eventresponse
    Content-Type: application/json

#### Parameters

**app_id** _required_
: The id of the originating app.

**user_id** _required_
: The dashboard user id. This will be sent to you by the Branch team to give you access to this API.

**calculation_type**  _required_
: This defines whether the rule can reward a user indefinitely, or a single time.

1. _0_ - Reward a user continually for the action.
1. _1_ - User is only eligible for single reward.

**location** _required_
: The user to reward for the action.

1. _0_ - The user completing the action receives credit.
1. _1_ - The user who referred the user completing the action receives credit.

**type** _required_
: The type of event response.

1. _"web_hook"_ - Register for a web hook callback when the criteria are met
1. _"credit"_ - For referral based rewards, reward the user who caused the referred install.
1. _"credit_session"_ - For referral based rewards, reward the user who referred the a new session.

**event** _required_ (max 63 characters)
: The event string to trigger the reward, eg "completed_purchase".

**metadata** _required_
: The metadata to define the event response, in JSON format.

For web hooks, use the following keys:

1. _"web_hook_url"_ - The url to call when an event occurs.

For credits, use the following keys;

1. _"amount"_ - the amount to reward the user
1. _"bucket"_ - the bucket to deposit the amount into

**filter** _optional_
: This is the set of keys and values that must be contained in the event metadata for this reward to be issued, in JSON format.

#### Returns

nothing

### Getting Current Branch App Config

#### Endpoint

  GET /v1/app/:app_id?user_id=[user id]

#### Parameters

**app_id** _required_
: The id of the originating app.

**user_id** _required_
: The dashboard user id. This will be sent to you by the Branch team to give you access to this API.

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


### Creating a New Branch App Config

#### Endpoint

    POST /v1/app
    Content-Type: application/json

#### Parameters

**user_id** _required_
: The dashboard user id. This will be sent to you by the Branch team to give you access to this API.

**app_name** _required_ (max 255 characters)
: The name of the app.

**dev_name** _required_ (max 255 characters)
: The main contact developer name.

**dev_email** _required_ (max 255 characters)
: The main contact developer email.

Note: we'll send an invite message to this email upon account creation.

**android_url** _optional_
: The url of the Android store, or namespace (com.android.myapp).

**android_uri_scheme** _optional_
: The Android URI scheme.

**ios_url** _optional_
: The url of iOS store, or app id (id512451233)

**ios_uri_scheme** _optional_
: The iOS URI scheme.

**ios_store_country** _optional_ (max 255 characters)
: The country code of the app, default to US.

**web_url** _optional_
: Backup website if URLs are null.

**text_message** _optional_ (max 255 characters)
: Text message to use for text-me feature, {{ link }} will be replaced with short link.

**og_app_id** _optional_ (max 255 characters)
: Default Open Graph (OG) app id.

**og_title** _optional_ (max 255 characters)
: Default OG title to be used with links.

**og_description** _optional_ (max 255 characters)
: Default OG description to be used with links.

**og_image_url** _optional_ (max 255 characters)
: Default OG image URL to be used with links.

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


### Updating a Branch App Config

#### Endpoint

    PUT /v1/app/:app_id
    Content-Type: application/json

#### Parameters

**app_id** _required_
: The id of the originating app.

**user_id** _required_
: The dashboard user id. This will be sent to you by the Branch team to give you access to this API.

**app_name** _optional_ (max 255 characters)
: The name of the app.

**dev_name** _optional_ (max 255 characters)
: The main contact developer name.

**dev_email** _optional_ (max 255 characters)
: The main contact developer email.

**android_url** _optional_
: The url of the Android store, or namespace (com.android.myapp).

**android_uri_scheme** _optional_
: The Android URI scheme.

**ios_url** _optional_
: The url of iOS store, or app id (id512451233)

**ios_uri_scheme** _optional_
: The iOS URI scheme.

**ios_store_country** _optional_ (max 255 characters)
: The country code of the app, default to US.

**web_url** _optional_
: Backup website if URLs are null.

**text_message** _optional_ (max 255 characters)
: The text message to use for text-me feature, {{ link }} will be replaced with short link.

**og_app_id** _optional_ (max 255 characters)
: Default Open Graph (OG) app id.

**og_title** _optional_ (max 255 characters)
: Default OG title to be used with links.

**og_description** _optional_ (max 255 characters)
: Default OG description to be used with links.

**og_image_url** _optional_ (max 255 characters)
: Default OG image URL to be used with links.

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


### Get/Create a Branch Referral Code

This API uses app_id and identity to retrieve a referral code; if none created yet, it uses the other params to create one and return it.

#### Endpoint

    POST /v1/referralcode
    Content-Type: application/json

#### Parameters

**app_id** _required_
: The id of the originating app.

**identity**  _required_ (max 127 characters)
: The referral code creator's identity.

**amount** _required (for create)_
: The amount of credit to redeem when user applies the referral code.

**bucket** _optional_ (max 63 characters)
: The name of the bucket to use. If none is specified, defaults to 'default'.

**expiration** _optional_
: The expiration date of the referral code.

**prefix** _optional_ (max 48 characters)
: The prefix to the referral code that you desire.

**calculation_type**  _required (for create)_
: This defines whether the referral code can be applied indefinitely, or only once per user.

1. _0_ - referral code can be applied continually
1. _1_ - a user can only apply a specific referral code once

**location** _required (for create)_
: The user to reward for applying the referral code.

1. _0_ - The user applying the referral code receives credit.
1. _2_ - The user who created the referral code receives credit.
1. _3_ - Both the user who created the referral code and the applying user receive credit.

#### Returns

  {
    referral_code: "The referral code. Without prefix, it's a 6 character long unique alpha-numeric string; with prefix, it's the prefix concatenated with a 4 character long unique alpha-numeric string",
    app_id: "The app key",
    metadata: {
      bucket: "The name of the bucket used for the referral code",
      amount: "The amount of the referral code",
    },
    expiration: "The expiration date of the referral code",
    calculation_type: "Whether the referral code can be applied indefinitely, or only once per user",
    location: "Whether to reward the creator of the referral code or the one what applies it"
  }

### Validate a Branch Referral Code

#### Endpoint

    POST /v1/referralcode/:code
    Content-Type: application/json

#### Parameters

**code** _required_
: The referral code to validate. NOTE: this param is passed via the URL structure.

**app_id** _required_
: The id of the originating app.

**identity**  _required_ (max 127 characters)
: The identity used to identify the user.

#### Returns

If the code is a valid referral code, and this user hasn't applied it in case of "unique" calculation_type, the response is:

  {
    referral_code: "The referral code. Without prefix, it's a 6 character long unique alpha-numeric string; with prefix, it's the prefix concatenated with a 4 character long unique alpha-numeric string",
    app_id: "The app key",
    metadata: {
      bucket: "The name of the bucket used for the referral code",
      amount: "The amount of the referral code",
    },
    expiration: "The expiration date of the referral code",
    calculation_type: "Whether the referral code can be applied indefinitely, or only once per user",
    location: "Whether to reward the creator of the referral code or the one what applies it"
  }

### Apply a Branch Referral Code

#### Endpoint

    POST /v1/applycode/:code
    Content-Type: application/json

#### Parameters

**code** _required_
: The referral code to apply. 
NOTE: this param is passed via the URL structure

**app_id** _required_
: The id of the originating app.

**identity**  _required_ (max 127 characters)
: The identity used to identify the user.

**user_id** _required_
: The dashboard user id. This will be sent to you by the Branch team to give you access to this API.

#### Returns

If the code is a valid referral code, and this user hasn't applied it in case of "unique" calculation_type, it returns the referral code JSONObject which includes the amount.

  {
    referral_code: "The referral code. Without prefix, it's a 6 character long unique alpha-numeric string; with prefix, it's the prefix concatenated with a 4 character long unique alpha-numeric string",
    app_id: "The app key",
    metadata: {
      bucket: "The name of the bucket used for the referral code",
      amount: "The amount of the referral code",
    },
    expiration: "The expiration date of the referral code",
    calculation_type: "Whether the referral code can be applied indefinitely, or only once per user",
    location: "Whether to reward the creator of the referral code or the one what applies it"
  }
