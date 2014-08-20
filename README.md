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

**tags** _optional_
: An array of strings, which are custom tags in which to categorize the links by. Recommended syntax: "tags":[t1,t2,t3]

**feature** _optional_
: the feature in which the link will be used. eg: "invite", "referral", "share", "gift", etc

**channel** _optional_
: the channel in which the link will be shared. eg: "facebook", "text_message"

**stage** _optional_
: A string value that represents the stage of the user in the app. eg: "level1", "logged_in", etc

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
