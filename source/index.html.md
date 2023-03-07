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

<aside class="notice">
For sake of simplicity, the docs will use <code>api.salesvista.app</code> as the domain for all sample endpoints, but you should use the domain associated with your organization's assigned region when accessing the API.
</aside>

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
const orgId = await svClient.getOrgId()
console.log('org id: ' + orgId)
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

Once you have been granted client credentials, you can use the [`client_credentials` oauth grant type](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/) to receive a bearer access token, which can be used to access data via specific HTTP endpoints.

<aside class="warning">
Make sure to keep your client secret <b>secret</b>! It should not be deployed with or exposed to client-side code. If you think your client secret may have been compromised, please contact SalesVista to revoke the old secret and receive a new one.
</aside>

The request body for oauth requests should use the `application/x-www-form-urlencoded` content type.

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
const org = await svClient.org.getOrg()
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

Parameter      | Type       | Default | Description
-------------- | ---------- | ------- | -----------
withCounts     | boolean    | false   | If set to true, the response will include entity counts across the org
from           | date       | none    | The start date to limit counts by e.g. `2023-01-01`
to             | date       | none    | The end date to limit counts by e.g. `2023-01-31`
fiscalYearId   | id or enum | current | Get counts for this fiscal year (instead of using from/to), supports pseudo-id values `current`, `previous`, and `next`
fiscalPeriodId | id or enum | none    | Get counts for this fiscal period (instead of using from/to), supports pseudo-id values `current`, `previous`, and `next`

# Sales

## List Sales

> Get a paginated list of sales:

```shell
# Plug in your own value for access_token and org_id
curl -H "Authorization: Bearer access_token" \
'https://api.salesvista.app/rest/v1/orgs/org_id/sales?page=1&size=500&fiscalPeriodId=current'
```

```javascript
// Your access token and org id will be stored in memory and used automatically.
const params = {
  page: 1,
  size: 500,
  fiscalPeriodId: 'current'
}
const sales = await svClient.sales.listSales(params)
console.log(JSON.stringify(sales, null, 2))
```

> Sample JSON response for a paginated list of sales:

```json
{
  "query": {
    "page": 1,
    "size": 500,
    "sort": [
      {
        "prop": "effectiveDate",
        "dir": "asc"
      },
      {
        "prop": "transactionDate",
        "dir": "asc"
      },
      {
        "prop": "transactionNum",
        "dir": "asc"
      }
    ],
    "fiscalPeriodId": "cozeyhrkrl9zpn4df000ohdnp"
  },
  "items": [
    {
      "id": "c20myllibl9zy1kv6006nvinp",
      "version": 0,
      "rep": {
        "id": "c6nz9hyh5l9zx06a1001rmqnp",
        "fullName": "rep1",
        "repCode": "rep1",
        "givenName": "Example",
        "familyName": "Rep",
        "title": null,
        "deleted": false,
        "deletedAt": null,
        "snapshotId": "cwmipm32jldkgijsr001kvuhv",
        "isInactive": false
      },
      "team": {
        "id": "c1fejnjqrl9zpojjo0029hdnp",
        "name": "Active Reps",
        "snapshotId": "cutufrgfkldkgijsz0022vuhv"
      },
      "productCategory": {
        "snapshotId": "c7ovps836ldkgijuo002uvuhv",
        "name": "Software",
        "id": "c7ovps836ldkgijuo002uvuhv"
      },
      "customerCategory": null,
      "product": {
        "id": "ckzdbytnfl9zqdc90008ahdnp",
        "name": "Software",
        "deleted": false,
        "deletedAt": null,
        "snapshotId": "cp4cbgroaldkgijsw001vvuhv",
        "productCode": "Software",
        "description": null,
        "displayName": "Software"
      },
      "customer": null,
      "contract": null,
      "effectiveDate": "2023-02-04",
      "transactionDate": "2023-02-04",
      "transactionNum": 100001,
      "annualContractValue": 0,
      "annualContractValueActual": 0,
      "annualContractValueCompensable": 0,
      "annualContractValueAttained": 0,
      "extendedAmount": 1052.8,
      "extendedAmountActual": 1052.8,
      "extendedAmountCompensable": 1052.8,
      "extendedAmountAttained": 1052.8,
      "grossMargin": null,
      "grossMarginActual": null,
      "grossMarginCompensable": null,
      "grossMarginAttained": null,
      "volume": 7,
      "volumeActual": 7,
      "volumeCompensable": 7,
      "volumeAttained": 7,
      "referenceId": "Some note",
      "status": "compensable",
      "type": "txn",
      "createdAt": "2023-03-02T17:59:43.842Z",
      "updatedAt": "2023-03-02T17:59:43.842Z",
      "reports": [
        {
          "id": "ce1itng39la03j67p000w1hnp",
          "slug": "15",
          "status": "closed",
          "name": "FEB",
          "stale": false
        }
      ],
      "labels": [],
      "territory__sv": [
        {
          "id": "cz5a504m2l9zxgp18002ivinp",
          "version": 0,
          "name": "Southeast"
        }
      ],
      "externalKey": null,
      "batch": {
        "id": "ce0gfoiral9zy1ktz006mvinp",
        "name": "SalesVista_Sales_1667411461586.csv",
        "deleted": false
      },
      "source": "file",
      "projectedCommissionWithDeferred": null,
      "projectedCompensationWithDeferred": null,
      "compensation": 0.86,
      "commission": 0.86,
      "bonus": 0,
      "deferredCommission": 0,
      "deferredCommissionPrereq": 0,
      "deferredCommissionEvent": 0,
      "commissionWithDeferred": 0.86,
      "compensationWithDeferred": 0.86
    }
  ],
  "matching": 1,
  "total": 284,
  "pages": 1,
  "more": false
}
```

Use this endpoint to fetch a paginated list of sales that exist in SalesVista for your organization.

Sales represent individual transactions (e.g. invoice line items) that are used as the input for performance-based compensation.

### Endpoint

`GET /rest/v1/orgs/:id/sales`

### Query Parameters

Parameter      | Type       | Default | Description
-------------- | ---------- | ------- | -----------
page           | integer    | 1       | The page number to fetch across matching results
size           | integer    | 50      | The number of sales to include per page
sort           | string     | effectiveDate,asc | The properties (and optional associated directions) to sort sales by e.g. `effectiveDate,desc,transactionNum,desc`
teamId         | id or array | none | Filter sales by one or more teams
includeChildTeams | boolean | false | Whether child teams should be included when filtering by team
productCategoryId | id or array | none | Filter sales by one or more product categories
customerCategoryId | id or array | none | Filter sales by one or more customer categories
labelId | id or array | none | Filter sales by one or more labels
repId | id or array | none | Filter sales by one or more reps/employees
id | id or array | none | Filter sales by one or more sale ids
productId | id or array | none | Filter sales by one or more products
customerId | id or array | none | Filter sales by one or more customers
from           | date       | none    | Filter sales by effective date using this start date by e.g. `2023-01-01`
to             | date       | none    | Filter sales by effective date using this end date by e.g. `2023-01-31`
fiscalYearId   | id or enum | none    | Filter sales by effective date using this fiscal year (instead of using from/to), supports pseudo-id values `current`, `previous`, and `next`
fiscalPeriodId | id or enum | none    | Filter sales by effective date using this fiscal period (instead of using from/to), supports pseudo-id values `current`, `previous`, and `next`
effectiveDate | date or array | none | Filter sales by one or more specific effective dates
effectiveDateAfter | date | none | Filter sales by effective date that are equal to or greater than this value
effectiveDateBefore | date | none | Filter sales by effective date that are equal to or less than this value
transactionDate | date or array | none | Filter sales by one or more specific transaction dates
transactionDateAfter | date | none | Filter sales by transaction date that are equal to or greater than this value
transactionDateBefore | date | none | Filter sales by transaction date that are equal to or less than this value
reportId | id or array | none | Filter sales by one or more compensation statements where the sales were processed and compensated for on these statements
batchId | id or array | none | Filter sales by one or more batches where the sales were created or updated in one of the given batches
referenceId | string | none | Filter sales by specific note value
noteLike | string | none | Filter sales by note where the note contains the given value
transactionNum | integer or array | none | Filter sales by one or more specific transaction numbers (human-friendly auto-generated incrementing number ids)
transactionNumLike | partial-integer string or array | none | Filter sales by transaction number where the transaction number on the sale contains the given partial value
source | string or array | none | Filter sales that came from one or more specific sources
externalKey | string or array | none | Filter sales by one or more specific external keys (typically ids defined in an external source system)
externalKeyLike | string or array | none | Filter sales by external key where the external key on the sale contains the given value
deferredAccrual | boolean | none | Only include sales that have deferred compensation tied to them
withWarnings | boolean | none | Only include sales that have warnings that need to be addressed by a user
status | enum or array | none | Filter sales by one or more specific statuses e.g. `compensable`
type | enum or array | none | Filter sales by one or more specific types e.g. `txn` or `churn`

## Get Sale Totals

> Get aggregate sale totals for a given period:

```shell
# Plug in your own value for access_token and org_id
curl -H "Authorization: Bearer access_token" \
'https://api.salesvista.app/rest/v1/orgs/org_id/sale-totals?fiscalPeriodId=current'
```

```javascript
// Your access token and org id will be stored in memory and used automatically.
const params = {
  fiscalPeriodId: 'current'
}
const sales = await svClient.sales.getSaleTotals(params)
console.log(JSON.stringify(sales, null, 2))
```

> Sample JSON response for aggregate sale totals:

```json
{
  "from": "2023-01-01",
  "to": "2023-01-31",
  "annualContractValueAttained": 40001000,
  "extendedAmountAttained": 47384458.22,
  "grossMarginAttained": 0,
  "volumeAttained": 136797,
  "annualContractValueActual": 40001000,
  "extendedAmountActual": 47384458.22,
  "grossMarginActual": 0,
  "volumeActual": 136797,
  "annualContractValueCompensable": 40001000,
  "extendedAmountCompensable": 47384458.22,
  "grossMarginCompensable": 0,
  "volumeCompensable": 136797,
  "numSales": 284
}
```

Get sale total aggregates for a given period or date range.

### Endpoint

`GET /rest/v1/orgs/:id/sale-totals`

### Query Parameters

Parameter      | Type       | Default | Description
-------------- | ---------- | ------- | -----------
from           | date       | none    | Filter sales by effective date using this start date by e.g. `2023-01-01`
to             | date       | none    | Filter sales by effective date using this end date by e.g. `2023-01-31`
fiscalYearId   | id or enum | none    | Filter sales by effective date using this fiscal year (instead of using from/to), supports pseudo-id values `current`, `previous`, and `next`
fiscalPeriodId | id or enum | none    | Filter sales by effective date using this fiscal period (instead of using from/to), supports pseudo-id values `current`, `previous`, and `next`
type | enum or array | none | Filter sales by one or more specific types e.g. `txn` or `churn`
