# Branch-Public-API

A public API to tie into for fancy integrations. All endpoints are appended to **https://api.branch.io**
___

## API Reference

1. URL Management
  + [Create a single link](#creating-a-deep-linking-url)
  + [Bulk create links](#bulk-creating-deep-linking-urls)
  + [Update an existing link](#modifying-existing-deep-linking-urls)
  + [Retrieve data from existing links](#viewing-state-of-existing-deep-linking-urls)
  + [Synchronous, query param link creation with app.link domains](#structuring-a-dynamic-deep-link-for-applink-domains)
  + [Synchronous, query param link creation with bnc.lt and custom domains](#structuring-a-dynamic-deep-link-for-bnclt-and-custom-domains)

2. Branch App Settings Management
  + [Retrieve existing app config](#getting-current-branch-app-config)
  + [Create a new app config](#creating-a-new-branch-app-config)
  + [Updating an app config](#updating-a-branch-app-config)

3. Credit Management
  + [Retrieve credit balance](#getting-credit-count)
  + [Add credits](#adding-credits)
  + [Redeem credits from a balance](#redeeming-credits)
  + [Reconciling potential fraud](#reconciling-credits)
  + [Retrieve a credit transaction history](#getting-the-credit-history)
  + [Create a reward rule or webhook dynamically](#creating-a-dynamic-reward-rule)

4. Custom Events
  + [Create a custom event via API](#creating-a-custom-event-for-funnels)
  
___

## Creating a Deep Linking URL

For more details on how to create links, see the [Branch link creation guide](https://dev.branch.io/link_creation_guide/)

#### Endpoint

    POST /v1/url
    Content-Type: application/json

#### Parameters

**branch_key** _required_
: The Branch key of the originating app.

##### Functional

**data** _optional_
: The dictionary to embed with the link. Accessed as session or install parameters from the SDK. **Use the data dictionary for all [link control parameters that you'll find here.](https://dev.branch.io/link_configuration/#redirect-customization)**

**alias** _optional_ (max 128 characters)
: Instead of our standard encoded short url, you can specify the alias of the link bnc.lt/alexaustin. Aliases are enforced to be unique per domain (bnc.lt, yourapp.com, etc). Be careful, link aliases are _unique_, immutable objects that cannot be deleted.

NOTE: If you POST to the this endpoint with the same alias, and a matching set of other POST parameters to an existing aliased link, the original will be returned to you. If it clashes and you don't specify a match, will return a HTTP 409 error.

**type** _optional_
: ADVANCED: 
- Set type to 1, to make the URL a one-time use URL. It won't deep link after 1 successful deep link.
- Set type to 2 to make a Marketing URL. These are URLs that are displayed under the Marketing tab on the dashboard (to also set the marketing title of the link, which shows up in the Marketing tab, set the *$marketing_title* field in the *data* dictionary to the value that you would like).
- *default* is set to 0, which is the standard Branch links created via our SDK.git

**duration** _optional_
: ADVANCED: In seconds. Only set this key if you want to override the match duration for deep link matching. This is the time that Branch allows a click to remain outstanding and be eligible to be matched with a new app session. This is default set to 7200 (2 hours)

##### Tracking

[**Branch analytics parameters**](https://dev.branch.io/link_configuration/#analytics-labels-for-data-organization) _optional_ It's important to tag your links with an organized structure of analytics labels so that the data appears consistent and readable in the dashboard.

**identity**  _optional_ (max 127 characters)
: The identity used to identify the user. If the link is not tied to an identity, there's no need to specify an identity.

#### Returns

```js
  {
    'url': 'http://bnc.lt/l/deeplink-randomID'
  }
```

#### Example

Here's an example cURL that you can paste into the terminal

```sh
curl -X POST \
\
-H "Content-Type: application/json" \
\
-d '{"branch_key":"key_live_feebgAAhbH9Tv85H5wLQhpdaefiZv5Dv", "campaign":"new_product_annoucement", "channel":"email", "tags":["monday", "test123"], "data":"{\"name\": \"Alex\", \"email\": \"alex@branch.io\", \"user_id\": \"12346\", \"$deeplink_path\": \"article/jan/123\", \"$desktop_url\": \"https://branch.io\"}"}' \
\
https://api.branch.io/v1/url
```

___

## Bulk creating Deep Linking URLs

For more details on how to create links, see the [Branch link creation guide](https://dev.branch.io/link_creation_guide/)

#### Endpoint

```sh
  POST /v1/url/bulk/:branch_key
  Content-Type: application/json
```

#### Parameters

A json array of pramameters from [Creating a Deep Linking URL.](https://dev.branch.io/link_creation_guide/) (Note: there is a 100KB limit on the request payload size)

```js
  [
   { 'channel': 'branch' },
   { 'channel': "fb", 'data': '{ "$og_title": "deep linking" }' }
  ]
```

#### Returns

An array of deep linking urls and/or errors in case invalid params.

```js
  [
   { 'url': 'http://bnc.lt/l/deeplink-randomID' },
   { 'error': 'error message' }  // in case of error
  ]
```
___

## Modifying Existing Deep Linking URLs

We've exposed an endpoint to update a certain category of Branch links through our API. Simply issue a PUT to our v1/url endpoint. Certain links may not be updated, which are /c/ and /d/ links. 

#### Endpoint
```sh
    PUT /v1/url?url=<url>
    Content-Type: application/json
```

#### Parameters

**branch_key** _required_
: The Branch key of the originating app. Found in the Branch Dashboard under **Settings**.

**branch_secret** _required_
: The paired Branch Secret to the Branch Key making the request. Found in the Branch Dashboard under **Settings**.

**url** _required_
: The URL you want to modify, including the host and domain, ex: https://bnc.lt/m/abcd1234, this is included on the URL to request itself:

    PUT https://api.branch.io/v1/url?url=https%3A%2F%2Fbnc.lt%2Ftest

**analytics/tracking parameters** _optional_

: [**Branch analytics parameters**](https://dev.branch.io/link_configuration/#analytics-labels-for-data-organization). Use these keys inside the body of the PUT (NOT inside the 'data' key) to modify data such as channel, campaign, tags, and more.

**data** _optional_

: The dictionary to embed with the link. Accessed as session or install parameters from the SDK. **Use the data dictionary for all [link control parameters that you'll find here.](https://dev.branch.io/link_configuration/#redirect-customization)**

#### Example

If you have a link with a URL of https://bnc.lt/test-link, a *channel* of 'facebook', and *data* of `{ "photo_id" : "50", "valid": "true" }` and want to update the channel, add an extra value to the dictionary, and add a campaign, here's how that would look:

    PUT https://api.branch.io/v1/url?url=https%3A%2F%2Fbnc.lt%2Ftest-link

    {
        "branch_key" : "key_live_xxxx",
        "branch_secret": "secret_live_xxxx",
        "channel": "twitter",
        "campaign": "twitter-november-campaign",
        "data": {
            "photo_id": "51",
            "photo_name": "John Smith",
            "$og_image_url": "https://imgur.com/abcd"
        }
    }

#### Returns

The new link returns existing data of the link plus the newly added data of the link. Following the example above, this is what would return.

```js
{
   "channel": "twitter",
   "campaign": "twitter-november-campaign",
   "feature": "share-button",
   "data": {
       "photo_id": "51",
       "valid": "true",
       "photo_name": "John Smith",
       "$og_image_url": "https://imgur.com/abcd",
       "~id": "123456789",
       "url": "https://bnc.lt/test-link"
   },
   "alias": "test-link",
   "type": 0
}
```

Note, some of this data is existing link data, and some is updated data, but the response will return all data.

#### Restrictions

There are certain restrictions when attempting to update links:
- Not all links are updateable, namely links with the structure of `bnc.lt/c/` or `bnc.lt/d/`
- The alias of a link cannot be updated, e.g. 'https://bnc.lt/test' -> 'https://bnc.lt/test1'
- The identity associated with a Branch link cannot be updated
- The `type` of a link cannot be changed, e.g. a marketing link is type 2, and a standard link generated by our Branch SDK is type 0
- The following additional fields cannot be updated:
  - `app_id`
  - `identity_id`
  - `domain`
  - `state`
  - `creation_source`
  - `app_short_identifier`
    
___

## Viewing State of Existing Deep Linking URLs

We've exposed an endpoint to view the contents of Branch links through our API. Simply issue a GET to our v1/url endpoint with the URL you'd like to view.

#### Endpoint
```sh
    GET /v1/url?url=<url>&branch_key=<branch key>
```

#### Parameters

**branch_key** _required_
: The Branch key of the originating app. Found in the Branch Dashboard under **Settings**.

**url** _required_
: The URL you want to modify, including the host and domain, ex: https://bnc.lt/m/abcd1234, this is included on the URL to request itself:

#### Example

If you have a link with a URL of `https://bnc.lt/m/7IhbRIjjmp` with Branch key of key_live_lceUuShIL0u4VHJv8BwEQmaitBfAXqCZ:

    GET https://api.branch.io/v1/url?url=https%3A%2F%2Fbnc.lt%2Fm%2F7IhbRIjjmp&branch_key=key_live_lceUuShIL0u4VHJv8BwEQmaitBfAXqCZ


#### Returns

The new link returns existing data of the link. Following the example above, this is what would return.

```js
{
   "channel": "twitter",
   "campaign": "twitter-november-campaign",
   "feature": "share-button",
   "data": {
       "photo_id": "51",
       "valid": "true",
       "photo_name": "John Smith",
       "$og_image_url": "https://imgur.com/abcd",
       "~id": "123456789",
       "url": "https://bnc.lt/test-link"
   },
   "alias": "test-link",
   "type": 0
}
```

---

## Structuring a 'dynamic' deep link for app.link domains

This should be used for situations where the longer link is okay and you want to create links quickly without a POST to the API. Here's a list of instructions on how to build a deep link: 

1. Start with your Branch domain, http://yourapp.app.link. 
2. [optional] Append the start of query params '?' 
3. [optional] Append the Branch analytics tag to keep your data organized in the dashboard. ([list here](https://dev.branch.io/getting-started/configuring-links/guide/#analytics-labels)) *channel=email&tags[]=drip1&tags[]=welcome*
4. [optional] Append any custom deep link parameters &user_id=4562&name=Alex&article_id=456
5. [optional] Append your Branch control parameters - see [a full list of them here](https://dev.branch.io/link_configuration/#redirect-customization)

#### Endpoint

```sh
  GET https://yourapp.app.link?AnyOptionalQueryParamsBelow
```

> Example:
https://branch.app.link?$deeplink_path=article%2Fjan%2F123&$fallback_url=https%3A%2F%2Fgoogle.com&channel=facebook&feature=affiliate

#### Parameters

For consistency, all parameters are kept in one spot since they are used for everything. Please find them in the [Link Configuration guide](https://dev.branch.io/getting-started/configuring-links/guide/)

---

## Structuring a 'dynamic' deep link for bnc.lt and custom domains

This should be used for situations where the longer link is okay and you want to create links quickly without a POST to the API. Here's a list of instructions on how to build a deep link: 

1. Start with your Branch domain, http://bnc.lt (or your white labeled one). 
2. Append /a/your_Branch_key.
3. [optional] Append the start of query params '?' 
4. [optional] Append the Branch analytics tag to keep your data organized in the dashboard. ([list here](https://dev.branch.io/getting-started/configuring-links/guide/#analytics-labels)) *channel=email&tags[]=drip1&tags[]=welcome*
5. [optional] Append any custom deep link parameters &user_id=4562&name=Alex&article_id=456
6. [optional] Append your Branch control parameters - see [a full list of them here](https://dev.branch.io/link_configuration/#redirect-customization)

#### Endpoint

```sh
  GET https://bnc.lt/a/<branch_key>?AnyOptionalQueryParamsBelow
```

> Example:
https://bnc.lt/a/key_live_jbgnjxvlhSb6PGH23BhO4hiflcp3y7ky?$deeplink_path=article%2Fjan%2F123&$fallback_url=https%3A%2F%2Fgoogle.com&channel=facebook&feature=affiliate

#### Parameters

For consistency, all parameters are kept in one spot since they are used for everything. Please find them in the [Link Configuration guide](https://dev.branch.io/getting-started/configuring-links/guide/)

___

## Getting Current Branch App Config

#### Endpoint

  GET /v1/app/[branch key]?branch_secret=[branch secret]

#### Parameters

**branch_key** _required_
: The id of the app to retrieve.

**branch_secret** _required_
: The secret of the app to retrieve.

#### Returns

```js
  {
    branch_key: "the app key",
    branch_secret: "the app secret",
    creation_date : "date app was created",

    app_name: "name of the app",

    dev_name: "main contact name",
    dev_email: "main contact email",
    dev_phone_number: "main contact phone",

    android_app: "whether an Android app is enabled",
    android_url: "url of Android store, or namespace (com.android.myapp)",
    android_uri_scheme: "the Android URI scheme",
    android_package_name: "the Android package name",
    sha256_cert_fingerprints: "the SHA256 fingerprints for App Links",
    android_app_links_enabled: "whether App Links are enabled",

    ios_app: "whether an iOS app is enabled",
    ios_url: "url of iOS store, or app id (id512451233)",
    ios_uri_scheme:  "the iOS URI scheme",
    ios_store_country: "the country code of the app, default to US",
    ios_bundle_id: "the iOS bundle ID",
    ios_team_id: "the iOS Team ID",
    universal_linking_enabled: "whether Universal Links are enabled",

    fire_url: "the redirect on Fire phones",
    windows_phone_url: "the redirect on Windows phones",
    blackberry_url: "The redirect on Blackberry phones",
    web_url: "backup website if URLs are null",
    default_desktop_url: "the default desktop redirect, or null if set to hosted SMS",

    short_url_domain: "white labeled domain for short links",

    text_message: "text message to use, {{ link }} will be replaced with short link",

    og_app_id: "optional default Open Graph (OG) app id",
    og_title: "optional default OG title",
    og_image_url: "optional default OG image URL",
    og_description: "optional default OG description",
    
    deepview_desktop: "the current deepview selected for the desktop platform",
    deepview_ios: "the current deepview selected for the iOS platform",
    deepview_android: "the current deepview selected for the Android platform",
  }
```

___

## Creating a New Branch App Config

#### Endpoint

```js
  POST /v1/app
  Content-Type: application/json
```

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

**android_app** _optional_
: Whether an Android app is enabled, (0 or 1 indicating present)

**android_url** _optional_
: The url of the Android store, or package name (com.android.myapp). `android_app` must be set to `2`.

**android_uri_scheme** _optional_
: The Android URI scheme.

**android_package_name** _optional_
: The Android package name (com.android.myapp)

**sha256_cert_fingerprints** _optional_
: The SHA256 fingerprints for App Links, in array form

**android_app_links_enabled** _optional_
: Whether App Links are enabled, (0 or 1 indicating true)

**ios_app** _optional_
: Whether an iOS app is enabled, (0 or 1 indicating present)

**ios_url** _optional_
: The url of iOS store, or app id (id512451233), or a fallback URL for iOS if present. `ios_app` must be set to `2`.

**ios_uri_scheme** _optional_
: The iOS URI scheme.

**ios_store_country** _optional_ (max 255 characters)
: The country code of the app, default to 'US'.

**ios_bundle_id** _optional_
: The iOS bundle ID

**ios_team_id** _optional_
: The iOS Team ID

**universal_linking_enabled** _optional_
: Whether Universal Links should be enabled, (0 or 1 indicating true)

**fire_url** _optional_
: The redirect on Fire phones

**windows_phone_url** _optional_
: The redirect on Windows phones

**blackberry_url** _optional_
: The redirect on Blackberry phones

**web_url** _optional_
: Backup website if URLs are null.

**default_desktop_url** _optional_
: The default desktop redirect, or null if set to hosted SMS

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

**deepview_desktop** _optional_
: The current deepview selected for the desktop platform, (eg "default", "my_template")

**deepview_ios** _optional_
: The current deepview selected for the iOS platform, (eg "default", "my_template")

**deepview_android** _optional_
: The current deepview selected for the Android platform, (eg "default", "my_template")

#### Request

```js
curl -XPOST -d '{
  "user_id": "...",
  "app_name": "eneff_test_3",
  "dev_name": "Ethan Neff",
  "dev_email": "eneff@branch.io",

  "always_open_app": "1",

  "android_app": "2", 
  "android_url": "https://www.example.com/ios", 
  "android_uri_scheme": "branchtest://", 
  "android_package_name": "com.branch.test", 
  "android_app_links_enabled": "1",  
  "sha256_cert_fingerprints": "sha256_cert_fingerprints": "14:6D:E9:83:C5:73:06:50:D8:EE:B9:95:2F:34:FC:64:16:A0:83:42:E6:1D:BE:A8:8A:04:96:B2:3F:CF:44:E5", 

  "ios_app": "2", 
  "ios_url": "https://www.example.com/ios", 
  "ios_uri_scheme": "branchtest://", 
  "ios_store_country": "US", 
  "universal_linking_enabled": "1",   
  "ios_bundle_id": "com.branch.test", 
  "ios_team_id": "PW4Q8885U7", 

  "fire_url": "https://www.example.com/amazon", 
  "windows_phone_url": "https://www.example.com/windows", 
  "blackberry_url": "https://www.example.com/blackberry", 
  "web_url": "https://www.example.com/web", 
  
  "default_desktop_url": "https://www.example.com/desktop", 
  "text_message": "click here to download",

  "og_app_id": "branch 123", 
  "og_title": "branch test", 
  "og_description": "branch description", 
  "og_image_url": "http://lorempixel.com/400/400/", 

  "deepview_desktop": "branch_default", 
  "deepview_ios": "branch_default", 
  "deepview_android": "branch_default"
}' 'https://api.branch.io/v1/app'
```

#### Response

```js
{
  "id": "...",
  "app_key": "...",
  "creation_date": "2016-12-01T19:38:25.661Z",
  "app_name": "eneff_test_3",
  "origin": "API: creator id = 293816316559643406, creator email = eneff@branch.io",
  "dev_name": "Ethan Neff",
  "dev_email": "eneff@branch.io",
  "always_open_app": "1",
  "android_app": "2",
  "android_url": "https://www.example.com/ios",
  "android_uri_scheme": "branchtest://",
  "android_package_name": "com.branch.test",
  "android_app_links_enabled": "1",
  "ios_app": "2",
  "ios_url": "https://www.example.com/ios",
  "ios_uri_scheme": "branchtest://",
  "ios_store_country": "US",
  "ios_bundle_id": "com.branch.test",
  "ios_team_id": "PW4Q8885U7",
  "universal_linking_enabled": "1",
  "fire_url": "https://www.example.com/amazon",
  "windows_phone_url": "https://www.example.com/windows",
  "blackberry_url": "https://www.example.com/blackberry",
  "web_url": "https://www.example.com/web",
  "default_desktop_url": "https://www.example.com/desktop",
  "short_url_domain": "",
  "default_short_url_domain": "94h3.app.link",
  "alternate_short_url_domain": "94h3-alternate.app.link",
  "text_message": "click here to download",
  "og_app_id": "branch 123",
  "og_title": "branch test",
  "og_image_url": "http://lorempixel.com/400/400/",
  "og_description": "branch description",
  "branch_key": "...",
  "branch_secret": "...",
  "deepview_desktop": "branch_default",
  "deepview_ios": "branch_default",
  "deepview_android": "branch_default"
}
```

___

## Updating a Branch App Config

#### Endpoint

```sh
  PUT /v1/app/:branch_key
  Content-Type: application/json
```

#### Parameters

**branch_key** _required_
: The id of the app to modify.

**branch_secret** _required_
: The branch secret of the app to modify.

**app_name** _optional_ (max 255 characters)
: The name of the app.

**dev_name** _optional_ (max 255 characters)
: The main contact developer name.

**dev_email** _optional_ (max 255 characters)
: The main contact developer email.

**android_url** _optional_
: The url of the Android store, or package name (com.android.myapp). Note that to set a fallback URL for Android instead, you must also set `android_app` to `2`.

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

```js
  {
    branch_key: "the app key",
    branch_secret: "the app secret",
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
```
___

## Getting Credit Count

#### Endpoint

```sh
  GET /v1/credits?branch_key=[branch key]&identity=[identity]
```

#### Parameters

**branch_key** _required_
: The Branch key of the originating app.

**identity**  _required_ (max 127 characters)
: The identity used to identify the user.

#### Returns

```sh
  {
    'default': 15,
    'other bucket': 4
  }
```

## Adding Credits

#### Endpoint

    POST /v1/credits
    Content-Type: application/json

#### Parameters

**branch_key** _required_
: The Branch key of the originating app.

**branch_secret** _required_
: The Branch secret of the originating app.

**identity**  _required_ (max 127 characters)
: The identity used to identify the user.

**amount** _required_
: The amount of credit to award.

**bucket** _optional_ (max 63 characters)
: The name of the bucket to use. If none is specified, defaults to 'default'.

#### Returns

`{ success: true }`

___

## Redeeming Credits

#### Endpoint

```sh
  POST /v1/redeem
  Content-Type: application/json
```

#### Parameters

**branch_key** _required_
: The Branch key of the originating app.

**branch_secret** _required_
: The Branch secret of the originating app.

**identity**  _required_ (max 127 characters)
: The identity used to identify the user.

**amount** _required_
: The amount of credit to redeem. Must be an integer.

**bucket** _optional_ (max 63 characters)
: The name of the bucket to use. If none is specified, defaults to 'default'.

#### Returns

Nothing if successful, or 402 error if not enough credits were available to redeem (this operation is [atomic](http://stackoverflow.com/questions/15054086/what-does-atomic-mean-in-programming), meaning if two callers try and redeem the same user's credits at the same time, only one will succeed).

___

## Reconciling Credits

If fraud is detected, e.g. users tricking the system to get more credits by referring him/herself, call this API to reconcile the amount from the user's credit balance. If the reconciliation amount is greater than the user's credit balance, the difference amount will be used and the credit balanced will be set to zero.

#### Endpoint

```sh
  POST /v1/reconcile
  Content-Type: application/json
```

#### Parameters

**branch_key** _required_
: The Branch key of the originating app.

**branch_secret** _required_
: The Branch secret of the originating app.

**identity**  _required_ (max 127 characters)
: The identity used to identify the user.

**amount** _required_
: The amount of credit to redeem.

**bucket** _optional_ (max 63 characters)
: The name of the bucket to use. If none is specified, defaults to 'default'.

#### Returns

The credit transaction JSON object for the reconciliation

```sh
  {
    "app_id": "55696551485375420",
    "identity_id": "55705482546906094",
    "date": "2015-02-13T02:27:12.477Z",
    "id": "94607584564086264",
    "bucket": "default",
    "type": 4,
    "amount": -5
  }
```

___

## Getting the Credit History

#### Endpoint

```sh
  GET /v1/credithistory?branch_key=[branch key]&identity=[identity]
```

#### Parameters

**branch_key** _required_
: The Branch key of the originating app.

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

```js
  [
    {
      "transaction": {
                       "date": "2014-10-14T01:54:40.425Z",
                       "id": "50388077461373184",
                       "bucket": "default",
                       "type": 0,
                       "amount": 5
                     },
      "event" : {
        "name": "event name",
        "metadata": { your event metadata if present }
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
      "event" : {
        "name": "event name",
        "metadata": { your event metadata if present }
      },
      "referrer": null,
      "referree": "12345678"
    }
  ]
```

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
3. _3_ or _5_ - This is a very unique case where we will subtract credits .automatically when we detect fraud.
4. _4_ - This is the type when you've called '/v1/reconcile' to reconcile credits manually

___

## Creating a Custom Event for Funnels

#### Endpoint

    POST /v1/event
    Content-Type: application/json

#### Parameters

**branch_key** _required_
: The Branch key of the originating app.

**branch_secret** _required_
: The Branch secret of the originating app.

**identity**  _required_ (max 127 characters)
: The identity used to identify the user.

**event** _required_ (max 63 characters)
: The event to associate with this identity.

**metadata** _optional_
:  Any associated parameters to be stored with the event. 1 layer JSON format. (max 255 characters for both keys and values)

#### Returns

nothing

___

## Creating a Dynamic Reward Rule

*Note:* Rules created via the API will not appear on the dashboard. This was done intentionally so as not to overwhelm the dashboard with automatically created rules.

#### Endpoint

```sh
    POST /v1/eventresponse
    Content-Type: application/json
```

#### Parameters

**branch_key** _required_
: The Branch key of the originating app.

**branch_secret** _required_
: The Branch secret of the originating app.

**calculation_type**  _required_
: This defines whether the rule can reward a user indefinitely, or a single time.

1. _0_ - Reward a user continually for the action.
1. _1_ - User is only eligible for single reward.

**location** _required_
: The user to reward for the action.

1. _0_ - Any user completing the action receives credit.
1. _1_ - The user who referred the user completing the action receives credit.
1. _4_ - The user completing the action *who was referred* by another user receives the credit.

**type** _required_
: The type of event response.

1. _"web_hook"_ - Register for a web hook callback when the criteria are met
1. _"credit"_ - For referral based rewards, reward the user who caused the referred install.

**event** _required_ (max 63 characters)
: The event string to trigger the reward, eg "completed_purchase".

**metadata** _required_
: The metadata to define the event response, in JSON format.

For web hooks, use the following keys:

1. _"web_hook_url"_ - The url to call when an event occurs. _must be a string_

For credits, use the following keys;

1. _"amount"_ - the amount to reward the user
1. _"bucket"_ - the bucket to deposit the amount into

**filter** _optional_
: This is the set of keys and values that must be contained in the event metadata for this reward to be issued, in JSON format.

#### Returns

nothing
