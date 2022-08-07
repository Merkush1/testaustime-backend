# Testaustime-rs api documentation

## General info

Testaustime API gives 5 different routes:
- [Auth](#auth)
- [Users](#users)
- [Activity](#activity)
- [Friends](#friends)
- [Leaderboards](#leaderboards)

Limits: 
- Usually Ratelimit: 10 req/m. The desired interval at which to send heartbeats is immediately when editing a file, and after that at max every 30 seconds, and only when the user does something actively in the editor.

## <a name="auth"></a>  Auth

Contains various user authorization operations

### Endpoints

| Endpoint|  Method | Description | 
| --- | --- | --- | 
| [/auth/register](#register) | POST | Creating a new user and returns the user auth token, friend code and registration_time | 
| [/auth/login](#login) | POST | Loging user to system and returns the user auth token and friend code | 
| [/auth/changeusername](#changeusername) | POST | Changing user username | 
| [/auth/changepassword](#changepassword) | POST | Changing user password | 
| [/auth/regenerate](#regenerate)  | POST | Regenerate user auth token | 

#### <a name="register"></a>    [1. POST /auth/register](#auth)

Creating a new user and returns the user auth token, friend code and registration_time. Ratelimit: 1 req/24h

**Header params:**

| Name |  Value | 
| --- | --- | 
| Content-Type | application/json | 

**Body params:**

| Param |  Type | Required | Description |
| --- | --- | --- | --- |
| username | string | Yes | Usename has to be between 2 and 32 characters long |
| password | string | Yes | Password has to be between 8 and 128 characters long |


**Sample request**
```curl
curl --request POST https://testaustime.fi/api/auth/register' \
--header 'Content-Type: application/json' \
--data-raw '{
    "username": "<username>",
    "password": "<password>"
}
```
**Sample response**
```JSON
{
    "auth_token": "<token>", 
    "username": "<username>", 
    "friend_code": "<friend_code>", 
    "registration_time": "YYYY-MM-DDTHH:MM:SS.sssssssssZ"
}
```

**Response definitions**
| Response Item | Type | Description | 
| --- | --- | --- | 
| auth_token | string | Bearer Auth token. Using for the all next resquests to identify user |
| username | string | Username |
| friend_code | string | By this code another users can add user to the friend list |
| registration_time | string (ISO 8601 format) | Time of registration to nanoseconds |


#### <a name="login"></a>  [2. POST /auth/login](#auth)

Logins to a users account returning the auth token

**Header params:**

| Name |  Value | 
| --- | --- | 
| Content-Type | application/json | 

**Body params:**

| Param |  Type | Required | Description |
| --- | --- | --- | --- |
| username | string | Yes | Usename has to be between 2 and 32 characters long |
| password | string | Yes | Password has to be between 8 and 128 characters long |


**Sample request**
```curl
curl --request POST 'https://testaustime.fi/api/auth/login' \
--header 'Content-Type: application/json' \
--data-raw '{
    "username": "<username>",
    "password": "<password>"
}'

```
**Sample response**
```JSON
{
    "id": 0,
    "auth_token": "<token>",
    "friend_code": "<friend_code>",
    "username": "<username>",
    "registration_time": "YYYY-MM-DDTHH:MM:SS.ssssssZ"
}
```

**Response definitions**
| Response Item | Type | Description | 
| --- | --- | --- | 
| id | int| User id |
| auth_token | string | Bearer Auth token. Using for the all next resquests to identify user |
| friend_code | string | By this code another users can add user to the friend list |
| username | string | Username |
| registration_time | string (ISO 8601 format) | Time of registration to microsends |

#### <a name="changeusername"></a>   [3. POST /auth/changeusername](#auth)

Changes username

**Header params:**

| Name |  Value | 
| --- | --- | 
| Content-Type | application/json |
| Authorization | Bearer `<token>` |

**Body params:**

| Param |  Type | Required | Description |
| --- | --- | --- | --- |
| new | string | Yes | New username. Usename has to be between 2 and 32 characters long |

**Sample request**
```curl
curl --request POST 'https://testaustime.fi/api/auth/changeusername' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <token> '{
    "new": "<new_username>"
}'
```
    
**Sample response**
```http
200 OK
```

**Error examples**
    
| Error | Error code | Body | 
| --- | --- | --- | 
| If "new" has <2 or >32 symbols | 400 Bad Request | `{"error" : "Username is not between 2 and 32 chars"}` | 
| If "new" is using existing username| 403 Forbidden | `"error"» : "User exists"` |

#### <a name="changepassword"></a>  [4. POST /auth/changepassword](#auth)

Changes users password

**Header params:**

| Name |  Value | 
| --- | --- | 
| Content-Type | application/json |
| Authorization | Bearer `<token>` |

**Body params:**

| Param |  Type | Required | Description |
| --- | --- | --- | --- |
| old | string | Yes | Current password |
| new | string | Yes | New password. Password has to be between 8 and 128 characters long |



**Sample request**
```curl
curl --request POST 'https://testaustime.fi/api/auth/changepassword' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <token>' \
--data-raw '{
   "old": "<old_password>",
   "new": "<new_password>"
}'
```
    
**Sample response**
```http
200 OK
```

**Error examples**
    
| Error | Error code | Body | 
| --- | --- | --- | 
| "new" has < 8 or >132 symbols  | 400 Bad Request | `{"error": "Password is not between 8 and 132 chars"}` | 
| "old" is incorrect| 401 Unathorized | `{"error": "You are not authorized"}` |
    
#### <a name="regenerate"></a>  [5. POST /auth/regenerate](#auth)

Regenerate users auth token
    
**Header params:**

| Name |  Value | 
| --- | --- | 
| Authorization | Bearer `<token>` |

**Sample request**
```curl
curl --request POST 'https://testaustime.fi/api/auth/regenerate' \
--header 'Content-Type: application/json'
```
    
**Sample response**
```JSON
{
    "token": "<token>"
}
```
    
**Response definitions**
    
| Response Item | Type | Description | 
| --- | --- | --- | 
| token | string| New Bearer Auth token. Using for the all next resquests to identify user |


## <a name="users"></a>  Users

Containts some operations with user and user friends


### Endpoints

| Endpoint|  Method | Description | 
| --- | --- | --- | 
| [/users/@me](#me) | GET | Geting data about authorized user | 
| [/users/@me/leaderboards](#my_leaderboards) | GET | Geting list of user leaderboards | 
| [/users/{username}/activity/data](#activity_data) | GET | Geting user or user friend coding activity data | 
| [/users/@me/delete](#delete_myself) | DELETE | Deliting user account  |

#### <a name="me"></a>  [1. GET /users/@me](#users)

Gets data about authorized user
    
**Header params:**

| Name |  Value | 
| --- | --- | 
| Authorization | Bearer `<token>` |

**Sample request**
```curl
curl --location --request GET 'https://testaustime.fi/api/users/@me' \
--header 'Authorization: Bearer `<token>`'
```
    
**Sample response**
```JSON
{
    "id": 0,
    "friend_code": "<friend_code>",
    "username": "<username>",
    "registration_time": "YYYY-MM-DDTHH:MM:SS.ssssssZ"
}
```
    
**Response definitions**
    
| Response Item | Type | Description | 
| --- | --- | --- | 
| id | int| User id |
| friend_code | string | By this code another users can add user to the friend list |
| username | string | Username |
| registration_time | string (ISO 8601 format) | Time of registration to microsends |

#### <a name="my_leaderboards"></a>  [2. GET /users/@me/leaderboards](#users)

Gets list of user leaderboards 
    
**Header params:**

| Name |  Value | 
| --- | --- | 
| Authorization | Bearer `<token>` |

**Sample request**
```curl
curl --location --request GET 'https://testaustime.fi/api/users/@me/leaderboards' \
--header 'Authorization: Bearer <token>'
```
    
**Sample response**
```JSON
[
    {
        "name": "Username's leaderboard",
        "member_count": 2
    }
]
```

**Response definitions**
    
| Response Item | Type | Description | 
| --- | --- | --- | 
| name | string | Name of leaderboard in which the user is a member |
| member_count | int | Number of users in the leaderboard |

#### <a name="activity_data"></a>  [3. GET /users/{username}/activity/data](#users)

Geting user or user friend coding activity data
    
**Header params:**

| Name |  Value | 
| --- | --- | 
| Authorization | Bearer `<token>` |

**Path params:**

| Path param |  Description | 
| --- | --- | 
| Username | Own or friend username. Also own username can be replaced on `@me`|

**Sample request**
```curl
curl --location --request GET 'https://testaustime.fi/api/users/@me/activity/data' \
--header 'Authorization: Bearer <token>'
```
    
**Sample response**
```JSON
[
    {
        "id": 0,
        "start_time": "YYYY-MM-DDTHH:MM:SS.ssssssZ",
        "duration": 0,
        "project_name": "string",
        "language": "string",
        "editor_name": "string",
        "hostname": "string"
    }
]
```

**Response definitions**
    
| Response Item | Type | Description | 
| --- | --- | --- | 
| id | int | ID of user code session |
| start_time | string (ISO 8601 format) | Start time (time of sending first heartbeat) of user code session to microsecnods | 
| duration | int | Duration of user code session in seconds |
| project_name | string| Name of the project in which user have a code session |
| language | string| Code language |
| editor_name | string| Name of IDA (Visual Studio Code, IntelliJ, Neovim, etc.) in which user is coding |
| hostname | string| User hostname |

#### <a name="delete_myself"></a>  [4. DELETE /users/@me/delete](#users)

Delites user account
    
**Header params:**

| Name |  Value | 
| --- | --- | 
| Content-Type | application/json |

**Body params:**

| Param |  Type | Required | Description |
| --- | --- | --- | --- |
| username| string | Yes | Username |
| password | string | Yes | User password |

**Sample request**
```curl
curl --request DELETE 'https://testaustime.fi/api/users/@me/delete' \
--header 'Content-Type: application/json' \
--data-raw '{
    "username": "<username>",
    "password": "<password>"
}'
```
    
**Sample response**
```http
200 OK
```

## <a name="activity"></a>  Activity

Contains main operations with activity heartbeats on which this service is based on

### Endpoints

| Endpoint|  Method | Description | 
| --- | --- | --- | 
| [/activity/update](#activity_up) | POST | Creating code session and logs current activity in that| 
| [/activity/flush](#activity_fl) | POST | Flushing any currently active coding session | 
| [/activity/delete](#activity_del) | DELETE | Deleting selected code session | 

#### <a name="activity_up"></a>  [1. POST /activity/update](#activity)

Main endpoint of the service. Creates code session and logs current activity in that. 

>*The desired interval at which to send heartbeats is immediately when editing a file, and after that at max every 30 seconds, and only when the user does something actively in the editor*
    
**Header params:**

| Name |  Value | 
| --- | --- | 
| Authorization | Bearer `<token>` |
| Content-Type | application/json |

**Body params:**

| Param | Type | Description | 
| --- | --- | --- | 
| language | string | Code language of the code session  |
| hostname | string | User hostname | 
| editor_name | string | Name of IDA (Visual Studio Code, IntelliJ, Neovim, etc.) in which user is coding |
| project_name | string| Name of the project in which user have a code session |


**Sample first request**

If the user doesn't have any active code session with this set of these values then first request `POST /activity/update` creates new code session

>Any other code session automatically stops after starting new one, so the user can't have >1 active code sessions in one time 

```curl
curl --request POST 'https://testaustime.fi/api/activity/update' \
--header 'Authorization: Bearer <token>' \
--header 'Content-Type: application/json' \
--data-raw '{
    "language": "Python",
    "hostname": "Hostname1",
    "editor_name": "IntelliJ",
    "project_name": "example_project"
}'
```
    
**Sample response** 
```HTTP
200 OK
Body: 0 
```

**Sample next request**

```curl
curl --request POST 'https://testaustime.fi/api/activity/update' \
--header 'Authorization: Bearer <token>' \
--header 'Content-Type: application/json' \
--data-raw '{
    "language": "Python",
    "hostname": "Hostname1",
    "editor_name": "IntelliJ",
    "project_name": "example_project"
}'
```
    
**Sample next response** 
```HTTP
200 OK
Body: PT7.420699439S //duration of the user code session in seconds to nanoseconds
```

#### <a name="activity_fl"></a>  [2. POST /activity/flush](#activity)

Flushes/stops any currently active coding session 

>*Active coding session can be flushed/stoped automatically without any activity updates for a long tinme (requests `POST /activity/update`*
    
**Header params:**

| Name |  Value | 
| --- | --- | 
| Authorization | Bearer `<token>` |

**Sample request**
```curl
curl --request POST 'https://testaustime.fi/api/activity/flush' \
--header 'Authorization: Bearer <token>'
```
    
**Sample response** 
```HTTP
200 OK
```

#### <a name="activity_del"></a>  [3. POST /activity/delete](#activity)

Deletes selected code session
    
**Header params:**

| Name |  Value | 
| --- | --- | 
| Authorization | Bearer `<token>` |

**Body params:**

| Param | Type | Description | 
| --- | --- | --- | 
| raw text | string | Maybe userid, maybe activity id, maybe something else, idk |

**Sample response** 
```HTTP
200 OK
```

**Error examples**
    
| Error | Error code | Body | 
| --- | --- | --- | 
| - | 404 Not Found | - | 


## <a name="friends"></a>  Friends

Containts CRUD-operations with user friends

### Endpoints

| Endpoint|  Method | Description | 
| --- | --- | --- | 
| [/friends/add](#add_friend) | POST | Adding the holder of the friend_token as a friend of the authenticating user | 
| [/friends/list](#list_friends) | GET | Geting a list of added user friends | 
| [/friends/regenerate](#regenerate_fc) | POST | Regenerateing the authorized user's friend code | 
| [/friends/remove](#remove_friend) | DELETE | Removing another user from your friend list |

#### <a name="add_friend"></a>  [1. POST /friends/add](#friends)

Adds the holder of the friend_token as a friend of the authenticating user
    
**Header params:**

| Name |  Value | 
| --- | --- | 
| Authorization | Bearer `<token>` |

**Body params:**

| Param | Type | Description | 
| --- | --- | --- | 
| raw text | string | Should contain <friend_code> without any prefixes |

**Sample request**

```curl
curl --request POST 'https://testaustime.fi/api/friends/add' \
--header 'Authorization: Bearer <token>' \
--data-raw '<friend_code>'
```
    
**Sample response** 
```HTTP
200 OK
```

**Error examples**
    
| Error | Error code | Body | 
| --- | --- | --- | 
| Friendcode is already used for adding a friend | 403 Forbidden | { "error": "Already friends"} |
| Friendcode from body request is not found | 404 Not Found | { "error": "User not found"} |
| Friendcode mathes with friendcode of authorized user himself the n 403 Forbidden | 403 Forbidden | { "error": "You cannot add yourself"} |

#### <a name="list_friends"></a>  [2. GET friends/list](#friends)

Gets a list of added user friends
    
**Header params:**

| Name |  Value | 
| --- | --- | 
| Authorization | Bearer `<token>` |

**Sample request**

```curl
curl --request GET 'https://testaustime.fi/api/friends/list' \
--header 'Authorization: Bearer <token>'
```
    
**Sample response** 
```JSON
[
    {
        "username": "<username>",
        "coding_time": {
            "all_time": 0,
            "past_month": 0,
            "past_week": 0
        }
    }
]
```

**Response definitions**
    
| Response Item | Type | Description | 
| --- | --- | --- | 
| username | string | Friend's username |
| coding_time | Object | Coding friend's time by total, past month and past week | 
| all_time | int | Total duration of user code sessions in seconds |
| past_month | int| Total duration of user code sessions in seconds for past month |
| past_week | int| Total duration of user code sessions in seconds for past week |

#### <a name="regenerate_fc"></a>  [3. POST /friends/regenerate](#friends)

Regenerates the authorized user's friend code
    
**Header params:**

| Name |  Value | 
| --- | --- | 
| Authorization | Bearer `<token>` |

**Sample request**
```curl
curl --request POST 'https://testaustime.fi/api/friends/regenerate' \
--header 'Authorization: Bearer <token>'
```
    
**Sample response**
```JSON
{
    "friend_code": "<friend_code>"
}
```
    
**Response definitions**
    
| Response Item | Type | Description | 
| --- | --- | --- | 
| friend_code | string| New friend code. Using for the all next create friends paire operations |

#### <a name="remove_friend"></a>  [4. DELETE /friends/remove](#friends)

Removes another user from your friend list
    
**Header params:**

| Name |  Value | 
| --- | --- | 
| Authorization | Bearer `<token>` |

**Body params:**

| Param | Type | Description | 
| --- | --- | --- | 
| raw text | string | friend's username should be without any prefixes |

**Sample response** 
```HTTP
200 OK
```

## <a name="leaderboards"></a>  Leaderboards

Containts CRUD-operations with leaderboards consisting of other Testaustime users

### Endpoints

| Endpoint|  Method | Description | 
| --- | --- | --- | 
| [/leaderboards/create](#create_lb) | POST | Adding new leaderboard | 
| [/leaderboard/join](#join_lb) | POST | Joining leaderboard by it's invite code | 
| [/leaderboards/{name}](#read_lb) | GET | Getting info about leaderboard if user is a member |
| [/leaderboard/{name}](#delete_lb) | DELETE | Deleting leaderboard if authorized user has admin rights |
| [/leaderboards/{name}/leave](#leave_lb) | POST | Leaving the leaderboard |
| [/leaderboards/{name}/regenerate](#regenerate_lb) | POST | Regenerating invite code of the leaderboard if authorized user has admin rights |
| [/leaderboards/{name}/promote](#promote_lb) | POST | Promoting member of a leaderboard to admin if authorized user has admin rights |
| [/leaderboards/{name}/demote](#demote_lb) | POST | Demoting promoted admin to regular member of the leaderboard if authorized user has root admin rights |
| [/leaderboards/{name}/kick](#kick_lb) | POST | Kicking user from leaderboard if authorized user has root admin rights |






