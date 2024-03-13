# Licorice partner integration - External Jan 2024


This document explains the two ways of integrating with Licorice to your website or app. The first one (and the most convenient) is to use the "@licoriceone/partner-lib" client-side library [available on npm](https://www.npmjs.com/package/@licoriceone/partner-lib). The second way is to use the API to implement the auth and data-fetching functionality by your own. The latter is meant for in-app integrations or server-to-server.

The Licorice partner integration uses OAuth 2.0 authorization protocol. It requires local authorization server. Licorice API uses Authorization Code grant type.

The client-side library is very tiny (1 kB) and is the most common way of integrating Licorice into your website.

Please email Roland@licorice.solutions

## Links to sections
[Go to Real Cool Heading section](#same-browser-window)
[Go to Real Cool Heading section](#same-browser-window-1)


## Client-side Library

### Installation

To install and save in your package.json a dependency, run the command below using npm:

```bash
npm install @licoriceone/partner-lib --save
```

or **yarn**:

```bash
yarn add @licoriceone/partner-lib
```

### Setup

> Looking for a complete vanilla JavaScript example? [Click here](#vanilla-javascript-example).

Initialize the library with the client ID you receive from the Licorice team:

```ts
import { initLicorice } from '@licoriceone/partner-lib'

const licorice = initLicorice({
  serverOrigin: values.serverOrigin,
  clientId: values.clientId,
})
```

### Add "Log in with Licorice" button

To handle the authentication process, you should call the `licorice.handleAuth` method on the `licorice` variable you've created in the previous step.

Calling this method will

1. Open a popup with the licorice website and the user will be prompted to continue to your website with his Licorice account or create a new Licorice account.
   OR
2. Open licorice website in the same tab and the user will be redirected to your website (URL you indicate) upon successful login.

#### Popup with JS Callback

```ts
licorice.handleAuth({
  name: 'Your website name',
  logo: 'https://your-website-logo', // This is scaled to 56x56px
  privacyText: {
    privacyTextOption: 'SELF', // Other options: USE_LICORICE, NONE
    privacyTextURI: 'https://your-website.com/privacy-text', // Leave undefined if privacyTextOption === 'USE_LICORICE' || privacyTextOption === 'NONE'
  },
  handler: {
    type: 'CALLBACK',
    onSuccess: () => {
      // handle success
    },
    onError: (error) => {
      // handle error
    },
  },
})
```

#### Same browser window

```ts
licorice.handleAuth({
  name: 'Your website name',
  logo: 'https://your-website-logo', // This is scaled to 56x56px
  privacyText: {
    privacyTextOption: 'SELF', // Other options: USE_LICORICE, NONE
    privacyTextURI: 'https://your-website.com/privacy-text', // Leave undefined if privacyTextOption === 'USE_LICORICE' || privacyTextOption === 'NONE'
  },
  handler: {
    type: 'REDIRECT',
    redirectURI: 'https://your-website.com/somepage',
  },
})
```

React example (Popup with JS Callback):

```tsx
const LicoriceAuthButton = () => {
  const [result, setResult] = useState<
    | { status: 'idle' }
    | { status: 'success' }
    | { status: 'error'; value: { errorCode: string } }
  >({ status: 'idle' })

  return (
    <button
      onClick={() => {
        licorice.handleAuth({
          name: 'Your website name',
          logo: 'https://your-website-logo',
          privacyText: {
            privacyTextOption: 'USE_LICORICE',
          },
          handler: {
            type: 'CALLBACK',
            onSuccess: () => {
              setResult({ status: 'success' })
            },
            onError: (error) => {
              setResult({ status: 'error', value: error })
            },
          },
        })
      }}
    >
      Log in with Licorice
    </button>
  )
}
```

React example (Same browser window):

```tsx
const LicoriceAuthButton = () => {
  return (
    <button
      onClick={() => {
        licorice.handleAuth({
          name: 'Your website name',
          logo: 'https://your-website-logo',
          privacyText: {
            privacyTextOption: 'USE_LICORICE',
          },
          handler: {
            type: 'REDIRECT',
            redirectURI: 'https://your-website.com/profile',
          },
        })
      }}
    >
      Log in with Licorice
    </button>
  )
}
```

### Vanilla JavaScript example

```html
<!-- index.html -->
<button type="button" id="licorice-button">Log in with Licorice</button>
<script src="src/licorice.js"></script>
```

#### Popup with JS Callback

```js
// index.js
import { initLicorice } from '@licoriceone/partner-lib'

const licorice = initLicorice({
  serverOrigin: '<insert your auth server origin here>',
  clientId: '<insert your client ID here>',
})

const button = document.getElementById('licorice-button')

button.onclick = () => {
  licorice.handleAuth({
    name: 'Partner name',
    logo: 'https://<partner_logo>',
    privacyText: { privacyTextOption: 'NONE' },
    handler: {
      type: 'CALLBACK',
      onSuccess: () => {
        // Here you can try to fetch user data fropm local authentication server
        console.log()
      },
      onError: (error) => {
        console.log(error) // Instance of JavaScript Error
      },
    },
  })
}
```

#### Same browser window

```js
// index.js
import { initLicorice } from '@licoriceone/partner-lib'

const licorice = initLicorice({
  serverOrigin: '<insert your auth server origin here>',
  clientId: '<insert your client ID here>',
})

const button = document.getElementById('licorice-button')

button.onclick = () => {
  licorice.handleAuth({
    name: 'Partner name',
    logo: 'https://<partner_logo>',
    privacyText: { privacyTextOption: 'USE_LICORICE' },
    handler: {
      type: 'REDIRECT',
      redirectURI: 'https://your-website.com/profile.html',
      state: '<random string>', // optional
    },
  })
}
```

### Client library API

#### `initLicorice`

```tsx
type InitLicoriceInput = Readonly<{
  serverOrigin: string
  clientId: string
}>

type LicoricePartnerOutput = Readonly<{
  handleAuth: (input: LicoricePartnerInput) => void
  destruct: () => void
}>

const initLicorice: ({
  serverOrigin,
  clientId,
}: InitLicoriceInput) => LicoricePartnerOutput
```

#### `LicoricePartnerInput`

```tsx
export type LicoricePartnerInput = {
  name: string
  logo: string
  privacyText?: PrivacyTextProps
  handler: HandleAuthRedirectInput | HandleAuthCallbackInput
}
```

#### `PrivacyTextProps`

```tsx
export type PrivacyTextProps =
  | {
      privacyTextOption: 'SELF'
      privacyTextURI: string
    }
  | { privacyTextOption: 'USE_LICORICE' }
  | undefined
```

#### `HandleAuthRedirectInput`

```tsx
export type HandleAuthRedirectInput = {
  type: 'REDIRECT'
  redirectURI: string
  state?: string
}
```

#### `HandleAuthCallbackInput`

```tsx
export type HandleAuthCallbackInput = {
  type: 'CALLBACK'
  popupWidth?: number
  popupHeight?: number
  onSuccess: (params: { token: string }) => void
  onError: (error: { errorCode: string }) => void
}
```

## Licorice server-side HTTP API

You will need to receive a `client id` and `client secret` from your Licorice account team in order to use this API.

### Validation

Resource for validating the partner's information :

Path:

```bash
POST https://licorice.me/api/v1/validation
```

Headers:

- `clientId: {{clientId}}`
- `reference: {{origin}}`

where:

- `{{clientId}}` is the client id received from your Licorice account team.
- `{{origin}}` is the origin (scheme, hostname, and port) that caused the request.

Response: `202 Accepted`

### User login

Path:

```bash
POST https://licorice.me/api/v1/authorization
```

Required query parameters:

- `state (string)` length of 10 symbols. A randomly generated unique value is typically used for this parameter
- `response_type (string)` must include `code`
- `client_id (string)` is the client id received from your Licorice account team.
- `ux_mode (string)` is the sign in mode `redirect` or `popup`.

Optional query parameters:

- `redirect_uri (string)` is the origin (scheme, hostname, and port) that caused the request.This param required if `ux_mode` equals `redirect`

Required headers:

- `reference (string)` is the origin (scheme, hostname, and port) that caused the request.

Body:

```json
{
  "password": "string",
  "username": "string"
}
```

where:

- `password` is a _required_ string at least 8 characters long, contain at least one number and one letter, max 90 characters
- `username` is a _required_ string max 80 characters

Response: `200 OK`

```json
{
  "code": "string",
  "state": "string",
  "expiredAt": "timestamp",
  "createdAt": "timestamp"
}
```

where:

- `code` is a _required_ unique temporary code
- `state` is a _required_ unique state which was provided in request
- `expiredAt` is a _required_ expiration date time for temporary code by UTC (note that code's lifetime equals 10 minutes)
- `createdAt` is a _required_ temporary code's creation date time by UTC

### Request an access token with a client_secret

Path:

```bash
POST https://licorice.me/api/v1/tokens
```

Required query parameters:

- `code (string)` temporary code that you acquired in the previous step.
- `client_id (string)` is the client id received from your Licorice account team.
- `grant_type (string)` must include `authorization_code`.
- `client_secret (string)` is the client secret received from your Licorice account team.
- `ux_mode (string)` is the sign in mode `redirect` or `popup`.

Optional query param:

- `redirect_uri (string)` is the origin (scheme, hostname, and port) that caused the request.This param required if `ux_mode` equals `redirect`

Required headers:

- `reference (string)` is the origin (scheme, hostname, and port) that caused the request.

Response: `200 OK`

```json
{
  "access_token": "string",
  "expired_at": "timestamp"
}
```

where:

- `access_token` is a _required_ the requested token which should be used for fetching user's data
- `expired_at` is a _required_ access token's expiration date time by UTC (note that access token's lifetime equals 30 minutes )

### User Data

Path:

```bash
GET https://licorice.me/api/v1/users/account
```

Headers:

- `token: {{accessToken}}`

where:

- `{{accessToken}}` is the access token that you acquired in the previous step

Response: `200 OK`

```json
{
  "externalId": "uuid",
  "preferences": {
    "age": "AGE_25_29",
    "country": "UK",
    "language": "EN",
    "localBusinesses": false,
    "thingsForOthers": false,
    "relatedToJob": false,
    "recentShopping": false,
    "zipCode": "string",
    "iabAudienceAgeId": "string",
    "iabCountry": "string"
  },
  "categories": [
    {
      "groupId": 0,
      "groupName": "string",
      "categoryName": "string",
      "categoryId": "uuid",
      "categoryCode": "string",
      "categoryExpanded": "string_budget",
      "habitsType": "BUDGET",
      "iabCategoryId": "string",
      "categoryIntegerId": "number"
    }
  ]
}
```

where:

- `externalId` is a _required_ UUID identifier of the user that can be used for external systems
- `preferences` is a _required_ object
- `age` can be `NO_AGE`, `AGE_LESS_13`, `AGE_13_17`, `AGE_18_20`, `AGE_21-24`, `AGE_25_29`, `AGE_30_34`, `AGE_35_39`, `AGE_40_44`, `AGE_45_49`, `AGE_50_54`, `AGE_55_59`, `AGE_60_64`, `AGE_65_69`, `AGE_70_74`, `AGE_MORE_75` or `''` when unset
- `country` is a _required_ [ISO 3166-1 alpha-2 country code](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) in uppercase
- `language` is a _required_ [ISO 639-1 language code](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) in uppercase
- `zipCode` is an _optional_ string max 80 characters, can be added only if `localBusinesses` is true
- `localBusinesses` is an _optional_ boolean
- `thingsForOthers` is an _optional_ boolean
- `relatedToJob` is an _optional_ boolean
- `recentShopping` is an _optional_ boolean
- `iabAudienceAgeId`: is a _required_ string that can be `''` in specific cases
- `iabCountry`: is a _required_ string
- `categories` is an _optional_ array of objects
- `groupId` is a _required_ numeric identifier of the group
- `groupName` is a _required_ name of the group
- `categoryId` is a _required_ UUID identifier of the category
- `categoryName` is a _required_ string
- `categoryCode` is a _required_ string that can be `''` when unset
- `categoryExpanded` is an _optional_ value (depends on the presence of `habitsType`) which is a concatenation of `categoryName` and `habitsType`
- `habitsType` is an can be `BUDGET`, `VALUE`, `QUALITY` or `''` when unset
- `iabCategoryId`: is a _required_ string
- `categoryIntegerId`: is a _required_ numeric identifier that starts from `1`

### Refresh an access token

Path:

```bash
POST https://licorice.me/api/v1/tokens/refresh
```

Response: `200 OK`

```json
{
  "accessToken": "string",
  "createdAt": "timestamp",
  "refreshTokenExpiredAt": "timestamp",
  "accessTokenExpiredAt": "timestamp"
}
```

where:

- `accessToken` is a _required_ the requested token which should be used for fetching user's data
- `createdAt` is a _required_ token's creation date time by UTC
- `refreshTokenExpiredAt` is a _required_ refresh token's expiration date time by UTC
- `accessTokenExpiredAt` is a _required_ access token's expiration date time by UTC (note that access token's lifetime equals 30 minutes )

### Error Responses

Server can respond 400+, 500+ codes with following response body:

```json
{
  "message": "string",
  "code": "string",
  "data": [
    {
      "key": "value"
    }
  ]
}
```

where:

- `message` is a _required_ string
- `code` is a _required_ string
- `data` is an _optional_ array of objects

---

For questions, please visit our website: https://licorice.solutions
