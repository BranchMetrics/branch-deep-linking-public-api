# Branch-Public-API

A public API to tie into for fancy integrations. All endpoints are appended to **https://api2.branch.io**
___

## API Reference

1. URL Management
  + [Create a single link](#creating-a-deep-linking-url)
  + [Bulk create links](#bulk-creating-deep-linking-urls)
  + [Update an existing link](#modifying-existing-deep-linking-urls)
  + [Retrieve data from existing links](#viewing-state-of-existing-deep-linking-urls)
  + [Synchronous, query param link creation with app.link domains](#structuring-a-dynamic-deep-link-for-applink-domains)
  + [Synchronous, query param link creation with bnc.lt and custom domains](#structuring-a-dynamic-deep-link-for-bnclt-and-custom-domains)

2. Logging Events
  + [Logging a commerce event (purchase, add to cart, etc)](#logging-commerce-events)
  + [Logging a content event (search, view content items, etc)](#logging-content-events)
  + [Logging a user lifecycle event (complete registration, unlock achievement, etc)](#logging-user-lifecycle-events)
  + [Logging other custom events](#logging-custom-events)

3. Branch App Settings Management
  + [Retrieve existing app config](#getting-current-branch-app-config)
  + [Create a new app config](#creating-a-new-branch-app-config)
  + [Updating an app config](#updating-a-branch-app-config)

4. Custom Events (deprecated)
  + [Create a custom events](#creating-custom-events)
  + [Create a custom commerce events](#creating-custom-commerce-events)
  
___

## Creating a Deep Linking URL

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
: Instead of our standard encoded short url, you can specify the alias of the link bnc.lt/devonaustin. Aliases are enforced to be unique per domain (bnc.lt, yourapp.com, etc). Be careful, link aliases are _unique_, immutable objects that cannot be deleted.

NOTE: If you POST to the this endpoint with the same alias, and a matching set of other POST parameters to an existing aliased link, the original will be returned to you. If it clashes and you don't specify a match, will return a HTTP 409 error.

**type** _optional_
: ADVANCED: 
- Set type to 1, to make the URL a one-time use URL. It won't deep link after 1 successful deep link.
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
-d '{"branch_key":"key_live_feebgAAhbH9Tv85H5wLQhpdaefiZv5Dv", "campaign":"new_product_annoucement", "channel":"email", "tags":["monday", "test123"], "data":"{\"name\": \"devon\", \"email\": \"devon@branch.io\", \"user_id\": \"12346\", \"$deeplink_path\": \"article/jan/123\", \"$desktop_url\": \"https://branch.io\"}"}' \
\
https://api2.branch.io/v1/url
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
   { "channel": "branch" },
   { "channel": "fb", "data": "{ \"$og_title\": \"deep linking\" }" }
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

    PUT https://api2.branch.io/v1/url?url=https%3A%2F%2Fbnc.lt%2Ftest

**analytics/tracking parameters** _optional_

: [**Branch analytics parameters**](https://dev.branch.io/link_configuration/#analytics-labels-for-data-organization). Use these keys inside the body of the PUT (NOT inside the 'data' key) to modify data such as channel, campaign, tags, and more.

**data** _optional_

: The dictionary to embed with the link. Accessed as session or install parameters from the SDK. **Use the data dictionary for all [link control parameters that you'll find here.](https://dev.branch.io/link_configuration/#redirect-customization)**

#### Example

If you have a link with a URL of https://bnc.lt/test-link, a *channel* of 'facebook', and *data* of `{ "photo_id" : "50", "valid": "true" }` and want to update the channel, add an extra value to the dictionary, and add a campaign, here's how that would look:

    PUT https://api2.branch.io/v1/url?url=https%3A%2F%2Fbnc.lt%2Ftest-link

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

    GET https://api2.branch.io/v1/url?url=https%3A%2F%2Fbnc.lt%2Fm%2F7IhbRIjjmp&branch_key=key_live_lceUuShIL0u4VHJv8BwEQmaitBfAXqCZ


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
4. [optional] Append any custom deep link parameters &user_id=4562&name=devon&article_id=456
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
5. [optional] Append any custom deep link parameters &user_id=4562&name=devon&article_id=456
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

## Logging Commerce Events

#### Endpoint

```js
  POST /v2/event/standard
  Content-Type: application/json
```

#### Parameters

Note about required identifiers. You must send up (in user_data):
1. developer_identity
OR
2. browser_fingerprint_id
OR
3. os=iOS AND (idfa OR idfv)
OR
4. os=Android AND (android_id or aaid)

--

*branch_key* _required_
: the app's branch_key

*name* _required_
: one of ADD_TO_CART, ADD_TO_WISHLIST, VIEW_CART, INITIATE_PURCHASE, ADD_PAYMENT_INFO, PURCHASE, SPEND_CREDITS

*user_data.os* _required_
: one of "Android", "iOS"

*user_data.os_version*
: version of the operating system. Specific to Android and iOS.

*user_data.environment* 
: usually FULL_APP. 

*user_data.aaid*
: Android/Google advertising id

*user_data.android_id*
: Android hardware id

*user_data.idfa*
: iOS advertising id

*user_data.idfv*
: iOS vendor id

*user_data.limit_ad_tracking* 
: true if the partner has opted to not be tracked by advertisers

*user_data.user_agent* 
: user agent of the browser or app where the event occurred. Usually associated with a webview.

*user_data.browser_fingerprint_id* 
: Branch internal-only field for tracking browsers

*user_data.http_origin* 
: current page url where Web SDK logged web session start

*user_data.http_referrer* 
: referral url that led to the current page where Web SDK logged web session start

*user_data.developer_identity* 
: developer-specified identity for a user

*user_data.country* 
: country code of the user, usually based on device settings or user agent string

*user_data.language* 
: language code of the user, usually based on device settings or user agent string

*user_data.local_ip* _Android only_
: local ip of the device

*user_data.brand* 
: brand of the device

*user_data.device_fingerprint_id* 
: Branch internal-only field for tracking devices

*user_data.app_version* 
: app version downloaded by the user

*user_data.model* 
: model of the device

*user_data.screen_dpi*
: screen's DPI

*user_data.screen_height*
: screen's height

*user_data.screen_width*
: screen's width

*custom_data*
: key-value pairs that the app developer would like attached to the event. Attached to events that are retrieved via Exports and sent via Webhooks.

*event_data.transaction_id*
: partner-specified transaction id for their internal use

*event_data.revenue*
: partner-specified reported revenue for the event

*event_data.currency*
: Currency that revenue, price, shipping, tax were orginally reported in by the partner

*event_data.shipping*
: Shipping cost associated with the transaction

*event_data.tax*
: Total tax associated with the transaction

*event_data.coupon*
: Transaction coupon redeemed with the transaction (e.g. "SPRING2017")

*event_data.affiliation*
: Store or affiliation from which this transaction occurred (e.g. Google Store)

*event_data.description*
: Description associated with the event, not necessarily specific to any individual content items (see below)

*content_items[i].$content_schema*
: category / schema for a piece of content. May be used in the future for analytics. One of:
  COMMERCE_AUCTION
  COMMERCE_BUSINESS
  COMMERCE_OTHER
  COMMERCE_PRODUCT
  COMMERCE_RESTAURANT
  COMMERCE_SERVICE
  COMMERCE_TRAVEL_FLIGHT
  COMMERCE_TRAVEL_HOTEL
  COMMERCE_TRAVEL_OTHER
  GAME_STATE
  MEDIA_IMAGE
  MEDIA_MIXED
  MEDIA_MUSIC
  MEDIA_OTHER
  MEDIA_VIDEO
  OTHER
  TEXT_ARTICLE
  TEXT_BLOG
  TEXT_OTHER
  TEXT_RECIPE
  TEXT_REVIEW
  TEXT_SEARCH_RESULTS
  TEXT_STORY
  TEXT_TECHNICAL_DOC

*content_items[i].$og_title*
: title (for the individual content item)

*content_items[i].$og_description*
: description (for individual content item)

*content_items[i].$og_image_url*
: image URL (for the individual content item)

*content_items[i].$canonical_identifier*
: used to allow Branch to unify content/messages for Content Analytics

*content_items[i].$publicly_indexable*
: `true`: content can be seen by anyone | `false`: cannot index for public use

*content_items[i].$locally_indexable*
: `true`: content can be indexed for local (device) use | `false`: cannot index for local use

*content_items[i].$price*
: price for the product/content

*content_items[i].$quantity*
: quantity of the item to be ordered (for PURCHASE, ADD_TO_CART, etc)

*content_items[i].$sku*
: product sku or product id

*content_items[i].$product_name*
: product's name

*content_items[i].$product_brand*
: product's brand

*content_items[i].$product_category*
: product's category, if it's a product. One of:
  ANIMALS_AND_PET_SUPPLIES
  APPAREL_AND_ACCESSORIES
  ARTS_AND_ENTERTAINMENT
  BABY_AND_TODDLER
  BUSINESS_AND_INDUSTRIAL
  CAMERAS_AND_OPTICS
  ELECTRONICS
  FOOD_BEVERAGES_AND_TOBACCO
  FURNITURE
  HARDWARE
  HEALTH_AND_BEAUTY
  HOME_AND_GARDEN
  LUGGAGE_AND_BAGS
  MATURE
  MEDIA
  OFFICE_SUPPLIES
  RELIGIOUS_AND_CEREMONIAL
  SOFTWARE
  SPORTING_GOODS
  TOYS_AND_GAMES
  VEHICLES_AND_PARTS

*content_items[i].$product_variant*
: product's variant (e.g. XL, red)

*content_items[i].$rating_average*
: average rating of the item

*content_items[i].$rating_count*
: number of ratings for the item

*content_items[i].$rating_max*
: maximum possible rating for the item (e.g. 5.0 if 5 stars is highest possible rating)

*content_items[i].$creation_timestamp*
: time the content was created

*content_items[i].$exp_date*
: the last time afterwhich this content is no longer valid. null / 0 mean no limit. Should rarely be set.

*content_items[i].$keywords*
: keywords

*content_items[i].$address_street*
: street address for a restaurant, business, room (hotel), etc

*content_items[i].$address_city*
: street address for a restaurant, business, room (hotel), etc

*content_items[i].$address_region*
: state or region for a restaurant, business, room (hotel), etc

*content_items[i].$address_country*
: country code for a restaurant, business, room (hotel), etc

*content_items[i].$address_postal_code*
: postal/zip code for a restaurant, business, room (hotel), etc

*content_items[i].$latitude*
: latitude for a restaurant, business, room (hotel), etc

*content_items[i].$longitude*
: longitude for a restaurant, business, room (hotel), etc

*content_items[i].$image_captions*
: captions associated with the image

*content_items[i].$condition*
: For auctions, whether the item is new, good, acceptable, etc. One of:
  OTHER
  NEW
  EXCELLENT
  GOOD
  FAIR
  POOR
  USED
  REFURBISHED

*content_items[i].$custom_fields*
: key-value pairs that the app developer would like attached to the content item. Attached to events that are retrieved via Exports and sent via Webhooks.

*metadata*
: internal use only


#### Request

```bash
curl -vvv -d '{
  "name": "PURCHASE",
  "customer_event_alias": "my custom alias",
  "user_data": {
    "os": "Android",
    "os_version": 25,
    "environment": "FULL_APP",
    "aaid": "abcdabcd-0123-0123-00f0-000000000000",
    "android_id": "a12300000000",
    "limit_ad_tracking": false,
    "developer_identity": "user123",
    "country": "US",
    "language": "en",
    "local_ip": "192.168.1.2",
    "brand": "LGE",
    "app_version": "1.0.0",
    "model": "Nexus 5X",
    "screen_dpi": 420,
    "screen_height": 1794,
    "screen_width": 1080
  },
  "custom_data": {
    "purchase_loc": "Palo Alto",
    "store_pickup": "unavailable"
  },
  "event_data": {
    "transaction_id": "tras_Id_1232343434",
    "currency": "USD",
    "revenue": 180.2,
    "shipping": 10.5,
    "tax": 13.5,
    "coupon": "promo-1234",
    "affiliation": "high_fi",
    "description": "Preferred purchase"
  },
  "content_items": [
    {
      "$content_schema": "COMMERCE_PRODUCT",
      "$og_title": "Nike Shoe",
      "$og_description": "Start loving your steps",
      "$og_image_url": "http://example.com/img1.jpg",
      "$canonical_identifier": "nike/1234",
      "$publicly_indexable": false,
      "$price": 101.2,
      "$locally_indexable": true,
      "$quantity": 1,
      "$sku": "1101123445",
      "$product_name": "Runner",
      "$product_brand": "Nike",
      "$product_category": "Sporting Goods",
      "$product_variant": "XL",
      "$rating_average": 4.2,
      "$rating_count": 5,
      "$rating_max": 2.2,
      "$creation_timestamp": 1499892854966,
      "$exp_date": 1499892854966,
      "$keywords": [
        "sneakers",
        "shoes"
      ],
      "$address_street": "230 South LaSalle Street",
      "$address_city": "Chicago",
      "$address_region": "IL",
      "$address_country": "US",
      "$address_postal_code": "60604",
      "$latitude": 12.07,
      "$longitude": -97.5,
      "$image_captions": [
        "my_img_caption1",
        "my_img_caption_2"
      ],
      "$condition": "NEW",
      "$custom_fields": "{\"foo1\":\"bar1\",\"foo2\":\"bar2\"}"
    },
    {
      "$og_title": "Nike Woolen Sox",
      "$canonical_identifier": "nike/5324",
      "$og_description": "Fine combed woolen sox for those who love your foot",
      "$publicly_indexable": false,
      "$price": 80.2,
      "$locally_indexable": true,
      "$quantity": 5,
      "$sku": "110112467",
      "$product_name": "Woolen Sox",
      "$product_brand": "Nike",
      "$product_category": "Apparel & Accessories",
      "$product_variant": "Xl",
      "$rating_average": 3.3,
      "$rating_count": 5,
      "$rating_max": 2.8,
      "$creation_timestamp": 1499892854966
    }
  ],
  "metadata": {},
  "branch_key": "key_test_hdcBLUy1xZ1JD0tKg7qrLcgirFmPPVJc"
}' https://api2.branch.io/v2/event/standard
```

#### Response

```bash
{ "branch_view_enabled": true/false }
```

___

## Logging Content Events

#### Endpoint

```js
  POST /v2/event/standard
  Content-Type: application/json
```

#### Parameters

Note about required identifiers. You must send up (in user_data):
1. developer_identity
OR
2. browser_fingerprint_id
OR
3. os=iOS AND (idfa OR idfv)
OR
4. os=Android AND (android_id or aaid)

--

*branch_key* _required_
: the app's branch_key

*name* _required_
: one of SEARCH, VIEW_ITEM, VIEW_ITEMS, RATE, SHARE

*user_data.os* _required_
: one of "Android", "iOS"

*user_data.os_version*
: version of the operating system. Specific to Android and iOS.

*user_data.environment* 
: usually FULL_APP. 

*user_data.aaid*
: Android/Google advertising id

*user_data.android_id*
: Android hardware id

*user_data.idfa*
: iOS advertising id

*user_data.idfv*
: iOS vendor id

*user_data.limit_ad_tracking* 
: true if the partner has opted to not be tracked by advertisers

*user_data.user_agent* 
: user agent of the browser or app where the event occurred. Usually associated with a webview.

*user_data.browser_fingerprint_id* 
: Branch internal-only field for tracking browsers

*user_data.http_origin* 
: current page url where Web SDK logged web session start

*user_data.http_referrer* 
: referral url that led to the current page where Web SDK logged web session start

*user_data.developer_identity* 
: developer-specified identity for a user

*user_data.country* 
: country code of the user, usually based on device settings or user agent string

*user_data.language* 
: language code of the user, usually based on device settings or user agent string

*user_data.local_ip* _Android only_
: local ip of the device

*user_data.brand* 
: brand of the device

*user_data.device_fingerprint_id* 
: Branch internal-only field for tracking devices

*user_data.app_version* 
: app version downloaded by the user

*user_data.model* 
: model of the device

*user_data.screen_dpi*
: screen's DPI

*user_data.screen_height*
: screen's height

*user_data.screen_width*
: screen's width

*custom_data*
: key-value pairs that the app developer would like attached to the event. Attached to events that are retrieved via Exports and sent via Webhooks.

*event_data.search_query*
: Search query associated with the event

*event_data.description*
: Description associated with the event, not necessarily specific to any individual content items (see below)

*content_items[i].$content_schema*
: category / schema for a piece of content. May be used in the future for analytics. One of:
  COMMERCE_AUCTION
  COMMERCE_BUSINESS
  COMMERCE_OTHER
  COMMERCE_PRODUCT
  COMMERCE_RESTAURANT
  COMMERCE_SERVICE
  COMMERCE_TRAVEL_FLIGHT
  COMMERCE_TRAVEL_HOTEL
  COMMERCE_TRAVEL_OTHER
  GAME_STATE
  MEDIA_IMAGE
  MEDIA_MIXED
  MEDIA_MUSIC
  MEDIA_OTHER
  MEDIA_VIDEO
  OTHER
  TEXT_ARTICLE
  TEXT_BLOG
  TEXT_OTHER
  TEXT_RECIPE
  TEXT_REVIEW
  TEXT_SEARCH_RESULTS
  TEXT_STORY
  TEXT_TECHNICAL_DOC

*content_items[i].$og_title*
: title (for the individual content item)

*content_items[i].$og_description*
: description (for individual content item)

*content_items[i].$og_image_url*
: image URL (for the individual content item)

*content_items[i].$canonical_identifier*
: used to allow Branch to unify content/messages for Content Analytics

*content_items[i].$publicly_indexable*
: `true`: content can be seen by anyone | `false`: cannot index for public use

*content_items[i].$locally_indexable*
: `true`: content can be indexed for local (device) use | `false`: cannot index for local use

*content_items[i].$price*
: price for the product/content

*content_items[i].$sku*
: product sku or product id

*content_items[i].$product_name*
: product's name

*content_items[i].$product_brand*
: product's brand

*content_items[i].$product_category*
: product's category, if it's a product. One of:
  ANIMALS_AND_PET_SUPPLIES
  APPAREL_AND_ACCESSORIES
  ARTS_AND_ENTERTAINMENT
  BABY_AND_TODDLER
  BUSINESS_AND_INDUSTRIAL
  CAMERAS_AND_OPTICS
  ELECTRONICS
  FOOD_BEVERAGES_AND_TOBACCO
  FURNITURE
  HARDWARE
  HEALTH_AND_BEAUTY
  HOME_AND_GARDEN
  LUGGAGE_AND_BAGS
  MATURE
  MEDIA
  OFFICE_SUPPLIES
  RELIGIOUS_AND_CEREMONIAL
  SOFTWARE
  SPORTING_GOODS
  TOYS_AND_GAMES
  VEHICLES_AND_PARTS

*content_items[i].$product_variant*
: product's variant (e.g. XL, red)

*content_items[i].$rating_average*
: average rating of the item

*content_items[i].$rating_count*
: number of ratings for the item

*content_items[i].$rating_max*
: maximum possible rating for the item (e.g. 5.0 if 5 stars is highest possible rating)

*content_items[i].$creation_timestamp*
: time the content was created

*content_items[i].$exp_date*
: the last time afterwhich this content is no longer valid. null / 0 mean no limit. Should rarely be set.

*content_items[i].$keywords*
: keywords

*content_items[i].$address_street*
: street address for a restaurant, business, room (hotel), etc

*content_items[i].$address_city*
: street address for a restaurant, business, room (hotel), etc

*content_items[i].$address_region*
: state or region for a restaurant, business, room (hotel), etc

*content_items[i].$address_country*
: country code for a restaurant, business, room (hotel), etc

*content_items[i].$address_postal_code*
: postal/zip code for a restaurant, business, room (hotel), etc

*content_items[i].$latitude*
: latitude for a restaurant, business, room (hotel), etc

*content_items[i].$longitude*
: longitude for a restaurant, business, room (hotel), etc

*content_items[i].$image_captions*
: captions associated with the image

*content_items[i].$condition*
: For auctions, whether the item is new, good, acceptable, etc. One of:
  OTHER
  NEW
  EXCELLENT
  GOOD
  FAIR
  POOR
  USED
  REFURBISHED

*content_items[i].$custom_fields*
: key-value pairs that the app developer would like attached to the content item. Attached to events that are retrieved via Exports and sent via Webhooks.

*metadata*
: internal use only


#### Request

```bash
curl -vvv -d '{
  "name": "VIEW_ITEMS",
  "customer_event_alias": "my custom alias",
  "user_data": {
    "os": "Android",
    "os_version": 25,
    "environment": "FULL_APP",
    "aaid": "abcdabcd-0123-0123-00f0-000000000000",
    "android_id": "a12300000000",
    "limit_ad_tracking": false,
    "developer_identity": "user123",
    "country": "US",
    "language": "en",
    "local_ip": "192.168.1.2",
    "brand": "LGE",
    "app_version": "1.0.0",
    "model": "Nexus 5X",
    "screen_dpi": 420,
    "screen_height": 1794,
    "screen_width": 1080
  },
  "custom_data": {
    "purchase_loc": "Palo Alto",
    "store_pickup": "unavailable"
  },
  "event_data": {
    "search_query": "red sneakers",
    "description": "Preferred purchase"
  },
  "content_items": [
    {
      "$content_schema": "COMMERCE_PRODUCT",
      "$og_title": "Nike Shoe",
      "$og_description": "Start loving your steps",
      "$og_image_url": "http://example.com/img1.jpg",
      "$canonical_identifier": "nike/1234",
      "$publicly_indexable": false,
      "$price": 101.2,
      "$locally_indexable": true,
      "$sku": "1101123445",
      "$product_name": "Runner",
      "$product_brand": "Nike",
      "$product_category": "Sporting Goods",
      "$product_variant": "XL",
      "$rating_average": 4.2,
      "$rating_count": 5,
      "$rating_max": 2.2,
      "$creation_timestamp": 1499892854966,
      "$exp_date": 1499892854966,
      "$keywords": [
        "sneakers",
        "shoes"
      ],
      "$address_street": "230 South LaSalle Street",
      "$address_city": "Chicago",
      "$address_region": "IL",
      "$address_country": "US",
      "$address_postal_code": "60604",
      "$latitude": 12.07,
      "$longitude": -97.5,
      "$image_captions": [
        "my_img_caption1",
        "my_img_caption_2"
      ],
      "$condition": "NEW",
      "$custom_fields": "{\"foo1\":\"bar1\",\"foo2\":\"bar2\"}"
    },
    {
      "$og_title": "Nike Woolen Sox",
      "$canonical_identifier": "nike/5324",
      "$og_description": "Fine combed woolen sox for those who love your foot",
      "$publicly_indexable": false,
      "$price": 80.2,
      "$locally_indexable": true,
      "$sku": "110112467",
      "$product_name": "Woolen Sox",
      "$product_brand": "Nike",
      "$product_category": "Apparel & Accessories",
      "$product_variant": "Xl",
      "$rating_average": 3.3,
      "$rating_count": 5,
      "$rating_max": 2.8,
      "$creation_timestamp": 1499892854966
    }
  ],
  "metadata": {},
  "branch_key": "key_test_hdcBLUy1xZ1JD0tKg7qrLcgirFmPPVJc"
}' https://api.branch.io/v2/event/standard
```

#### Response

```bash
{ "branch_view_enabled": true/false }
```
___

## Logging User Lifecycle Events

#### Endpoint

```js
  POST /v2/event/standard
  Content-Type: application/json
```

#### Parameters

Note about required identifiers. You must send up (in user_data):
1. developer_identity
OR
2. browser_fingerprint_id
OR
3. os=iOS AND (idfa OR idfv)
OR
4. os=Android AND (android_id or aaid)

--

*branch_key* _required_
: the app's branch_key

*name* _required_
: one of SEARCH, VIEW_CONTENT, VIEW_CONTENT_LIST, RATE, SHARE_CONTENT_ITEM

*user_data.os* _required_
: one of "Android", "iOS"

*user_data.os_version*
: version of the operating system. Specific to Android and iOS.

*user_data.environment* 
: usually FULL_APP. 

*user_data.aaid*
: Android/Google advertising id

*user_data.android_id*
: Android hardware id

*user_data.idfa*
: iOS advertising id

*user_data.idfv*
: iOS vendor id

*user_data.limit_ad_tracking* 
: true if the partner has opted to not be tracked by advertisers

*user_data.user_agent* 
: user agent of the browser or app where the event occurred. Usually associated with a webview.

*user_data.browser_fingerprint_id* 
: Branch internal-only field for tracking browsers

*user_data.http_origin* 
: current page url where Web SDK logged web session start

*user_data.http_referrer* 
: referral url that led to the current page where Web SDK logged web session start

*user_data.developer_identity* 
: developer-specified identity for a user

*user_data.country* 
: country code of the user, usually based on device settings or user agent string

*user_data.language* 
: language code of the user, usually based on device settings or user agent string

*user_data.local_ip* _Android only_
: local ip of the device

*user_data.brand* 
: brand of the device

*user_data.device_fingerprint_id* 
: Branch internal-only field for tracking devices

*user_data.app_version* 
: app version downloaded by the user

*user_data.model* 
: model of the device

*user_data.screen_dpi*
: screen's DPI

*user_data.screen_height*
: screen's height

*user_data.screen_width*
: screen's width

*custom_data*
: key-value pairs that the app developer would like attached to the event. Attached to events that are retrieved via Exports and sent via Webhooks.

*event_data.description*
: Description associated with the event, not necessarily specific to any individual content items (see below)

*metadata*
: internal use only


#### Request

```bash
curl -vvv -d '{
  "name": "COMPLETE_REGISTRATION",
  "user_data": {
    "os": "Android",
    "os_version": 25,
    "environment": "FULL_APP",
    "aaid": "abcdabcd-0123-0123-00f0-000000000000",
    "android_id": "a12300000000",
    "limit_ad_tracking": false,
    "developer_identity": "user123",
    "country": "US",
    "language": "en",
    "local_ip": "192.168.1.2",
    "brand": "LGE",
    "app_version": "1.0.0",
    "model": "Nexus 5X",
    "screen_dpi": 420,
    "screen_height": 1794,
    "screen_width": 1080
  },
  "custom_data": {
    "foo": "bar"
  },
  "event_data": {
    "description": "Preferred purchase"
  },
  "metadata": {},
  "branch_key": "key_test_hdcBLUy1xZ1JD0tKg7qrLcgirFmPPVJc"
}' https://api.branch.io/v2/event/standard
```

#### Response

```bash
{ "branch_view_enabled": true/false }
```

___

## Logging Custom Events

#### Endpoint

```js
  POST /v2/event/custom
  Content-Type: application/json
```

#### Parameters

Note about required identifiers. You must send up (in user_data):
1. developer_identity
OR
2. browser_fingerprint_id
OR
3. os=iOS AND (idfa OR idfv)
OR
4. os=Android AND (android_id or aaid)

--

*branch_key* _required_
: the app's branch_key

*name* _required_
: string for custom event name

*user_data.os* _required_
: one of "Android", "iOS"

*user_data.os_version*
: version of the operating system. Specific to Android and iOS.

*user_data.environment* 
: usually FULL_APP. 

*user_data.aaid*
: Android/Google advertising id

*user_data.android_id*
: Android hardware id

*user_data.idfa*
: iOS advertising id

*user_data.idfv*
: iOS vendor id

*user_data.limit_ad_tracking* 
: true if the partner has opted to not be tracked by advertisers

*user_data.user_agent* 
: user agent of the browser or app where the event occurred. Usually associated with a webview.

*user_data.browser_fingerprint_id* 
: Branch internal-only field for tracking browsers

*user_data.http_origin* 
: current page url where Web SDK logged web session start

*user_data.http_referrer* 
: referral url that led to the current page where Web SDK logged web session start

*user_data.developer_identity* 
: developer-specified identity for a user

*user_data.country* 
: country code of the user, usually based on device settings or user agent string

*user_data.language* 
: language code of the user, usually based on device settings or user agent string

*user_data.local_ip* _Android only_
: local ip of the device

*user_data.brand* 
: brand of the device

*user_data.device_fingerprint_id* 
: Branch internal-only field for tracking devices

*user_data.app_version* 
: app version downloaded by the user

*user_data.model* 
: model of the device

*user_data.screen_dpi*
: screen's DPI

*user_data.screen_height*
: screen's height

*user_data.screen_width*
: screen's width

*custom_data*
: key-value pairs that the app developer would like attached to the event. Attached to events that are retrieved via Exports and sent via Webhooks.

*metadata*
: internal use only


#### Request

```bash
curl -vvv -d '{
  "name": "picture swiped",
  "customer_event_alias": "my custom alias",
  "user_data": {
    "os": "Android",
    "os_version": 25,
    "environment": "FULL_APP",
    "aaid": "abcdabcd-0123-0123-00f0-000000000000",
    "android_id": "a12300000000",
    "limit_ad_tracking": false,
    "developer_identity": "user123",
    "country": "US",
    "language": "en",
    "local_ip": "192.168.1.2",
    "brand": "LGE",
    "app_version": "1.0.0",
    "model": "Nexus 5X",
    "screen_dpi": 420,
    "screen_height": 1794,
    "screen_width": 1080
  },
  "custom_data": {
    "foo": "bar"
  },
  "metadata": {},
  "branch_key": "key_test_hdcBLUy1xZ1JD0tKg7qrLcgirFmPPVJc"
}' https://api.branch.io/v2/event/custom
```

#### Response

```bash
{ "branch_view_enabled": true/false }
```

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
: The dashboard user id. Can be found on your [Branch Dashboard](https://dashboard.branch.io/settings/account)

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

```bash
curl -XPOST -d '{
  "user_id": "...",
  "app_name": "eneff_test_3",
  "dev_name": "...",
  "dev_email": "...",

  "always_open_app": "1",

  "android_app": "2", 
  "android_url": "https://www.example.com/ios", 
  "android_uri_scheme": "branchtest://", 
  "android_package_name": "com.branch.test", 
  "android_app_links_enabled": "1",  
  "sha256_cert_fingerprints": ["14:6D:E9:83:C5:73:06:50:D8:EE:B9:95:2F:34:FC:64:16:A0:83:42:E6:1D:BE:A8:8A:04:96:B2:3F:CF:44:E5"], 

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

```bash
{
  "id": "...",
  "app_key": "...",
  "creation_date": "2016-12-01T19:38:25.661Z",
  "app_name": "eneff_test_3",
  "origin": ...",
  "dev_name": "...",
  "dev_email": "...",
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

**branch_secret** _required_
: The branch secret of the app to modify.

**app_name** _optional_ (max 255 characters)
: The name of the app.

**dev_name** _required_ (max 255 characters)
: The main contact developer name.

**dev_email** _required_ (max 255 characters)
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

## Creating Custom Events

*DEPRECATED*

This endpoint, /v1/event, is deprecated in favor of /v2/event. Please refer to these sections instead:
  + [Logging a commerce event (purchase, add to cart, etc)](#logging-commerce-events)
  + [Logging a content event (search, view content items, etc)](#logging-content-events)
  + [Logging a user lifecycle event (complete registration, unlock achievement, etc)](#logging-user-lifecycle-events)
  + [Logging other custom events](#logging-custom-events)

#### Endpoint

```sh
POST /v1/event
Content-Type: application/json
```

#### Parameters

- **branch_key** _required_
: The Branch key of the originating app.

- **identity**  _required_ (max 127 characters)
: The identity used to identify the user.

- **event** _required_ (max 63 characters)
: The event to associate with this identity.

- **metadata** _optional_
:  Any associated parameters to be stored with the event. 1 layer JSON format. (max 255 characters for both keys and values)

#### Testing

- [Branch Dashboard Liveview](https://dashboard.branch.io/liveview/events)

#### Example

```sh
curl --request POST \
  --url https://api.branch.io/v1/event \
  --header 'cache-control: no-cache' \
  --header 'content-type: application/json' \
  --data '{
    "branch_key": "...",
    "identity": "360202634827023565",
    "event": "done",
    "metadata": {
      "hello": "world",
      "custom_data": "this"
    }
  }'
```

#### Returns

```sh
{}
```

___

## Creating Custom Commerce Events

*DEPRECATED*

This endpoint, /v1/event, is deprecated in favor of /v2/event. Please refer to these sections instead:
  + [Logging a commerce event (purchase, add to cart, etc)](#logging-commerce-events)
  + [Logging a content event (search, view content items, etc)](#logging-content-events)
  + [Logging a user lifecycle event (complete registration, unlock achievement, etc)](#logging-user-lifecycle-events)
  + [Logging other custom events](#logging-custom-events)

#### Endpoint

```sh
POST /v1/event
Content-Type: application/json
```

#### Parameters

- **branch_key** _required_
: The Branch key of the originating app.

- **identity**  _required_ (max 127 characters)
: The identity used to identify the user.

- **event** _required_ (max 63 characters)
: The event to associate with this identity.

- **metadata** _optional_
:  Any associated parameters to be stored with the event. 1 layer JSON format. (max 255 characters for both keys and values)

- **commerce_data**
:  Any commerce parameters to be stored with the event. (max 255 characters for both keys and values)

#### Testing

- [Branch Dashboard Liveview](https://dashboard.branch.io/liveview/commerce_events)


#### Example

```sh
curl --request POST \
  --url https://api.branch.io/v1/event \
  --header 'cache-control: no-cache' \
  --header 'content-type: application/json' \
  --data '{
    "branch_key": "...",
    "identity": "360202634827023565",
    "event": "purchase",
    "metadata": {
      "hello": "world",
      "custom_data": "this"
    },
    "commerce_data": {
      "revenue": 50.0,
      "currency": "USD",
      "transaction_id": "foo-transaction-id",
      "shipping": 0.0,
      "tax": 5.0,
      "affiliation": "foo-affiliation",
      "products": [
        { 
          "sku": "foo-sku-1",
          "name": "foo-item-1",
          "price": 45.00,
          "quantity": 1,
          "brand": "foo-brand",
          "category": "Electronics",
          "variant": "foo-variant-1"
        },
        { 
          "sku": "foo-sku-2",
          "price": 2.50,
          "quantity": 2
        }
      ]
    }
  }'
```

#### Returns

```sh
{}
```
