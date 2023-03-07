# Errors

The following common error codes are used across all endpoints:


Code | Name | Meaning
---- | ---- | -------
400  | Bad Request | There is a problem with the request that must be addressed by the client.
401  | Unauthorized | Your access token was not provided or is expired.
402  | Payment Required | Your organization's subscription is not in good standing.
403  | Forbidden | Your client is not authorized to make this request.
404  | Not Found | The resource requested does not exist.
405  | Method Not Allowed | This HTTP method is not allowed for this URL/route.
429  | Too Many Requests | Your client has exceeded its allowed API limits.
500  | Internal Server Error | An unexpected error occurred within SalesVista. Please try again later or contact SalesVista for help.
503  | Service Unavailable | The SalesVista API is temporarily offline for maintenance. Please try again later.
504  | Gateway Timeout | The API is processing your request but took too long to respond.
