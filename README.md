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

### Creating a dynamic reward rule

#### Endpoint

    POST /v1/eventresponse

#### Parameters

**app_id** _required_
: The id of the originating app

**calculation_type**  _required_
: This defines whether the rule can reward a user indefinitely, or a single time.

1. 0 - reward a user continually for the action
1. 1 - user is only eligible for single reward

**location** _required_
: The user to reward for the action

1. 0 - the user completing the action receives credit
1. 1 - the user who referred the user completing the action receives credit

**type** _required_
: the type of event response

1. "web_hook" - register for a web hook callback when the criteria are met
1. "credit" - for referral based rewards, reward the user who caused the referred install
1. "credit_session" - for referral based rewards, reward the user who referred the a new session

**event** _required_
: The event string to trigger the reward, eg "completed_purchase"

**metadata** _required_
: The metadata to define the event response, in JSON format.

For web hooks, use the following keys:

1. "web_hook_url" - the url to call when an event happens

For credits, use the following keys;

1. "amount" - the amount to reward the user
1. "bucket" - the bucket to deposit the amount into

**filter** _optional_
: This is the set of keys and values that must be contained in the event metadata for this reward to be issued, in JSON format.

#### Returns

nothing
