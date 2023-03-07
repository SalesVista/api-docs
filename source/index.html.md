---
title: SalesVista API Reference

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers
  - shell
  - javascript

toc_footers:
  - <a href='https://github.com/SalesVista/api-client-node'>View the Node.js SDK</a>
  - <a href='https://salesvista.app/signin'>Sign in to SalesVista</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the SalesVista API
---

# Introduction

Welcome to the SalesVista API! This docs site is meant to help you use the SalesVista API to access sales and compensation data for your organization.

Throughout the docs, there are language examples for curl (shell) and Node.js (javascript). You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right or using the nav menu on smaller screens.

# Assigned Region

SalesVista hosts your organization's data within one of several production environments. Each environment represents an "assigned region" and is hosted at a different domain.

For sake of simplicity, the docs will use `api.salesvista.app` as the domain for all sample endpoints, but you should use the domain associated with your organization's assigned region when accessing the API.

If you do not know your organization's assigned region or the domain associated with your assigned region, please contact your SalesVista customer success representative.

# Authentication

> Use your client credentials to get an access token:

```shell
# Get an access token using your client id and secret
curl https://api.salesvista.app/oauth/v1/token \
  -u 'client_id:client_secret' \
  -d 'grant_type=client_credentials'

# Use the access token and org id
# Plug in your own value for access_token and org_id
curl https://api.salesvista.app/rest/v1/orgs/org_id \
  -H "Authorization: Bearer access_token"
```

```javascript
const svClient = require('@salesvista/client')

svClient.configure({
  // your client credentials
  id: process.env.SALESVISTA_CLIENT_ID,
  secret: process.env.SALESVISTA_CLIENT_SECRET,
  // use the domain associated with your org's assigned region
  baseUrl: 'https://api.salesvista.app'
})

// You can manually get an access token via the following, but note
// the SDK library will automatically manage access tokens for you
// when you use normal API endpoints, so the following is not necessary.
// It is just for illustrative purposes.
const token = await svClient.getToken()
console.log('access token: ' + token.access_token)
console.log('org id: ' + token.org_id)
```

> An example JSON response for an access token:

```json
{
  "access_token": "xtSab92x5vSDacQanmCfUjuAXx41ix0vTMioThzyhAyy9w",
  "expires_in": 172800,
  "org_id": "cf69g0r2cl9zpn4b9000ihdnp",
  "token_type": "Bearer"
}
```

The SalesVista API uses OAuth2 for authentication.

Before you can use the API, you must have a registered oauth client. At this time, oauth clients must be registered manually by the SalesVista team. To request API access, please contact your SalesVista customer success representative or send an email to tech@salesvista.com.

Once you have been granted client credentials, you can use the `client_credentials` oauth grant type to receive a bearer access token, which can be used to access data via specific HTTP endpoints.

<aside class="notice">
Make sure to keep your client secret secret! It should not be deployed with or exposed to client-side code. If you think your client secret may have been compromised, please contact SalesVista to revoke the old secret and receive a new one.
</aside>

The access token response body will be in JSON and will contain an `access_token` property and an `org_id` property. Use the `access_token` value in the `Authorization` request header for every subsequent request. Use the `org_id` value when necessary when calling specific endpoints.

When using the Node.js SDK, access tokens will be automatically managed for you. They will be automatically fetched when necessary and stored in memory. If you want to customize how tokens are stored, you can specify your own `cacheStrategy`. See the [SDK docs](https://github.com/SalesVista/api-client-node#auth-token-storage) for more info.

# Org

## Get Org

> Get basic org info:

```shell
# Plug in your own value for access_token and org_id
curl https://api.salesvista.app/rest/v1/orgs/org_id \
  -H "Authorization: Bearer access_token"
```

```javascript
// Your access token and org id will be stored in memory and used automatically.
const org = await svClient.getOrg()
console.log(JSON.stringify(org, null, 2))
```

> Sample JSON response for an org (without counts):

```json
{
  "id": "cf69g0r2cl9zpn4b9000ihdnp",
  "version": 3,
  "orgName": "example",
  "companyName": "Example",
  "currency": "USD",
  "fyStart": "01-01",
  "aggregationInterval": "monthly",
  "fiscalYearUseStartDate": true,
  "currentWorkingFiscalYear": {
    "id": "cmdd4fp2bl9zpn4cg000khdnp",
    "name": "FY2022",
    "start": "2023-01-01",
    "end": "2023-12-31"
  }
}
```

This endpoint fetches information about your organization within SalesVista.

### Endpoint

`GET /rest/v1/orgs/:id`

### Query Parameters

Parameter      | Default | Description
-------------- | ------- | -----------
withCounts     | false   | If set to true, the response will include entity counts across the org.
from           | none    | The start date to limit counts by.
to             | none    | The end date to limit counts by.
fiscalYearId   | none    | Get counts for this fiscal year (instead of using from/to).
fiscalPeriodId | none    | Get counts for this fiscal period (instead of using from/to).
