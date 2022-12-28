# okta-client-secret-rotation-workflow
A OKTA workflow solution to allow authorized API consumers with a valid client id and client secret to rotate their secret online without any administrator involvement. Also offers ability for consumers to delete the old secret using their existing valid client credential.

The Workflow exposes a invocation URL and offers the following 2 APIs.

###### Prerequisites :
1. OKTA Workflows must be enabled for the tenant (Okta offers 5 free workflows with a paid teannt in preview and production) 
   write to support or ask your CSM to enable this
2. OKTA connection set up in workflow

###### Working:
1. Import the workflow and try it out on a test consumer created in the org.
2. Add the client IDs of authorized clients in the workflow table - "Authorized Clients"
3. Consumers must know their current secretID, clientID and ClientSecret and workflow invocation URL to call the APIs

###### Notes : 
1. A consumer(client) can have at most two client_secret at a time, if rotate is called when there are already 2 active client secrets an error is returned.

### Rotate existing client secret - Request
```
curl --location --request POST 'https://yourtenant.workflows.oktapreview.com/api/flo/7f3xxxxxxxxxxx3d04f95073a3b441/invoke?clientToken=80f52750xxxxxxxxxxxxxxxxxxxbe106c9b59e49' \
--header 'Content-Type: application/json' \
--data-raw '{
  "clientId" : "0oa5xira1KBe3d7",
  "currentClientSecret" : "P-uVzjmI6PGzhiUAafw9ro-cSSy_w8vX9AG",
  "secretId" : "ocs5xuira8MR1d7",
  "mode" : "rotate"
}'
```
#### Request Parameter Details

| Parameter Name | Mandatory    |   Description |
| :---         |     :---       |  :---          |
| clientId   | Yes     | Existing valid clientId of the consumer  |
| currentClientSecret     | Yes       | Existing client_secret of consumer      |
| secretId | Yes | SecretId of the current secret |
| mode | Yes | “rotate” for generating a new secret|

#### Rotate existing client secret - Response
```
HTTP/1.1 200 OK
Content-Type: application/json
{
    "client_secret": "3GtKmaaZS9wKTzqNvsLcqtguSOtikx",
    "secretId": "ocs61a83KwI6",
    "clientId": "0oa5xira1KBe3d7"
}
```
#### Response Parameter details

| Parameter Name    |   Description |
| :---              |  :---          |
| clientId    | clientId of the consumer  |
| secretId  | SecretId of the new secret generated  |
| client_secret | New client secret generated |

#### Delete old secretId - Request
```
curl --location --request POST 'https://yourtenant.workflows.oktapreview.com/api/flo/7f3xxxxxxxxxxx3d04f95073a3b441/invoke?clientToken=80f52750xxxxxxxxxxxxxxxxxxxbe106c9b59e49' \
--header 'Content-Type: application/json' \
--data-raw '{
  "clientId" : "0oa5xira1KBe3d7",
  "currentClientSecret" : "3GtKmaaZS9wKTzqNvsLcqtguSOtikx",
  "secretId" : "ocs5xuira8MR1d7",
  "mode" : "delete"
}'
```
#### Request Parameter Details

| Parameter Name | Mandatory    |   Description |
| :---         |     :---       |  :---          |
| clientId   | Yes     | Existing valid clientId of the consumer  |
| currentClientSecret     | Yes       | Existing client_secret of consumer [Use new one generated]      |
| secretId | Yes | SecretId of the secret you want to delete (old secretID) |
| mode | Yes | "delete” for inactivating old secret|

#### Rotate existing client secret - Response
```
HTTP/1.1 200 OK
Content-Type: application/json
{
    "status": "INACTIVE",
    "secretId": "ocs5xuira8MR1d7"
}
```
#### Response Parameter details

| Parameter Name    |   Description |
| :---              |  :---          |
| secretId  | SecretId of secret inactivated  |
| status | status of the secret inactivated |

#### Error Responses
1. Required parameters are not provided
```
HTTP/1.1 400
Content-Type: application/json
{
    "error": "In Valid Parameters"
}
```
2. When Client ID is not authorized for rotation/deletion, or Source IP is not authorized (if configured)
```
HTTP/1.1 403
Content-Type: application/json
{
    "error" : "Un-Authorized"
}
```

3. Invalid clientID or secret
```
HTTP/1.1 403
Content-Type: application/json
{
  "error" : "InValid ClientId/Secret"
}
```

4. Rate Limit error - When there are too many requests for the client (5 requests / 15 mins)
```
HTTP/1.1 403
Content-Type: application/json
{
  "error" : "Limit Exceeded"
}
```

5. Other Errors 
```
HTTP/1.1 500
Content-Type: application/json
{
"error" : "UnExpected Error Occurred, Pl contact administrator"
}
```

6. Run time errors
```
HTTP/1.1 5XX or 4xx
Content-Type: application/json
{
    "_error": true,
    "retry_count": 0,
    "flo": "okta:1.0.592:hTTPRequest",
    "method": "ggMnPBF_lAT",
    "execution": "2a251c7c-c111-4f30-b5a5-4e8c8ea759cb",
    "module": "http.call",
    "kind": "HTTP Request Error",
    "statusCode": 400,
    "headers": {
        "content-type": "application/json",
        "x-content-type-options": "nosniff",
        "public-key-pins-report-only": "pin-sha256=\"jZomPEBSDXoipA9un78hKRIeN/+U4ZteRaiX8YpWfqc=\"; pin-sha256=\"axSbM6RQ+19oXxudaOTdwXJbSr6f7AahxbDHFy3p8s8=\"; pin-sha256=\"SE4qe2vdD9tAegPwO79rMnZyhHvqj3i5g1c2HkyGUNE=\"; pin-sha256=\"ylP0lMLMvBaiHn0ihLxHjzvlPVQNoyQ+rMiaj0da/Pw=\"; max-age=60; report-uri=\"https://okta.report-uri.com/r/default/hpkp/reportOnly\"",
        "x-okta-request-id": "Y2Tm8ij6PgwPQAbYI_1x0gAAD1c",
        "strict-transport-security": "max-age=315360000; includeSubDomains",
        "server": "nginx",
        "cache-control": "no-cache, no-store",
        "content-security-policy": "default-src 'self' xxxx.oktapreview.com *.oktacdn.com; connect-src 'self' xxx.oktapreview.com xxx-admin.oktapreview.com *.oktacdn.com *.mixpanel.com *.mapbox.com app.pendo.io data.pendo.io pendo-static-5634101834153984.storage.googleapis.com pendo-static-5391521872216064.storage.googleapis.com xxx.kerberos.oktapreview.com https://oinmanager.okta.com data:; script-src 'unsafe-inline' 'unsafe-eval' 'self' xxxx.oktapreview.com *.oktacdn.com; style-src 'unsafe-inline' 'self' xxx.oktapreview.com *.oktacdn.com app.pendo.io cdn.pendo.io pendo-static-5634101834153984.storage.googleapis.com pendo-static-5391521872216064.storage.googleapis.com; frame-src 'self' xxxx.oktapreview.com xxxx-admin.oktapreview.com login.okta.com; img-src 'self' xxxx.oktapreview.com *.oktacdn.com *.tiles.mapbox.com *.mapbox.com app.pendo.io data.pendo.io cdn.pendo.io pendo-static-5634101834153984.storage.googleapis.com pendo-static-5391521872216064.storage.googleapis.com data: blob:; font-src 'self' xxxx.oktapreview.com data: *.oktacdn.com fonts.gstatic.com; frame-ancestors 'self'",
        "x-xss-protection": "0",
        "expect-ct": "report-uri=\"https://oktaexpectct.report-uri.com/r/t/ct/reportOnly\", max-age=0",
        "date": "Fri, 04 Nov 2022 10:18:27 GMT",
        "transfer-encoding": "chunked",
        "x-rate-limit-reset": "1667557164",
        "p3p": "CP=\"HONK\"",
        "expires": "0",
        "x-rate-limit-remaining": "98",
        "connection": "keep-alive",
        "pragma": "no-cache",
        "set-cookie": [
            "sid=\"\"; Expires=Thu, 01-Jan-1970 00:00:10 GMT; Path=/",
            "autolaunch_triggered=\"\"; Expires=Thu, 01-Jan-1970 00:00:10 GMT; Path=/",
            "JSESSIONID=E2E5D3578876EF6203221943C0ECFDCE; Path=/; Secure; HttpOnly"
        ],
        "x-rate-limit-limit": "100"
    },
    "body": {
        "errorCode": "E0000001",
        "errorSummary": "Api validation failed: OAuth2ClientSecretMediated",
        "errorLink": "E0000001",
        "errorId": "oae4f2ugrhlTtKZPYN3M61Mrg",
        "errorCauses": [
            {
                "errorSummary": "You have reached the maximum number of client secrets per client."
            }
        ]
    },
    "message": "400 Bad Request",
    "code": 400,
    "description": "HTTP Request Error",
    "steps": 26,
    "source": {
        "flo": "okta:1.0.592:hTTPRequest",
        "method": "ggMnPBF_lAT",
        "execution": "2a251c7c-c111-4f30-b5a5-4e8c8ea759cb",
        "module": "http.call"
    },
    "error": {
        "message": "400 Bad Request"
    }
}
```

Note: The execution ID can help your team Admin to trace back the failed request and suggest corrective action in case of a issue in API call response.
