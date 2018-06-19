# Errors

TicketJames API uses the following error codes:


Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request is invalid.
401 | Unauthorized -- Your credentials are wrong.
403 | Forbidden
404 | Not Found
405 | Method Not Allowed -- You tried to access an invalid method.
406 | Not Acceptable -- You requested a format that isn't json.
410 | Gone -- The requested object has been removed from our servers.
418 | I'm a teapot.
429 | Too Many Requests -- You're making too many requests! Slow down!
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- We're temporarily offline for maintenance. Please try again later.
