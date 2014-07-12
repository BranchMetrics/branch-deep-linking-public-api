Branch-Public-API
=================

a public API to tie into for fancy integrations. All endpoints are appended to **http://api.branchmetrics.io**

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

#### Parameters

**app_id** _required_
: The id of the originating app

**identity**  _required_
: The identity used to identify the user.

**amount** _required_
: The amount of credit to redeem.

**bucket** _optional_
: The name of the bucket to use. If none is specified, defaults to 'default'

#### Returns

Nothing if successful, or 402 error if not enough credits were available to redeem (this operation is atomic, meaning if two callers try and redeem the same user's credits at the same time, only one will succeed).

### Creating a deeplinking URL

#### Endpoint

    POST /v1/url

#### Parameters

**app_id** _required_
: The id of the originating app

**identity**  _required_
: The identity used to identify the user.

**tag** _optional_
: A tag for splitting out data in the dashboard. 

**data** _optional_
: The dictionary to embed with the link. Accessed as session or install parameters from the SDK

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
: Set type to 1, to make the URL a one-time use URL. It won't deep link after 1 successful link

#### Returns

    {
        'url': 'http://bnc.lt/l/deeplink-randomID'
    }
