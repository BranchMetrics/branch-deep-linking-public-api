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

**data** _optional_
: The dictionary to embed with the link. Accessed as session or install parameters from the SDK

**Note** 
You can customize the Facebook OG tags of each URL if you want to dynamically share content by using the following optional keys in the params dictionary:
    "$og_app_id"
    "$og_title"
    "$og_description"
    "$og_image_url"

**tags** _optional_
: An array of strings, which are custom tags in which to categorize the links by. Recommended syntax: "tags":[t1,t2,t3]

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