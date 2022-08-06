# Testaustime-rs api documentation

## General info

Testaustime API gives 5 different routes:
- [auth](#auth)
- [users](#users)
- activity
- friends
- leaderboards

Limits: 
- Usually Ratelimit: 10 req/m. The desired interval at which to send heartbeats is immediately when editing a file, and after that at max every 30 seconds, and only when the user does something actively in the editor.

## <a name="auth"></a>  auth

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
    "registration_time": "YYYY-DD-MMTHH:MM:SS.sssssssssZ"
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
    "registration_time": "YYYY-DD-MMTHH:MM:SS.ssssssZ"
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


## <a name="users"></a>  users

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
    "registration_time": "YYYY-DD-MMTHH:MM:SS.ssssssZ"
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
        "start_time": "YYYY-DD-MMTHH:MM:SS.ssssssZ",
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

**Error examples**
    





    
    

    






























    
    

    




































    
    

    

































