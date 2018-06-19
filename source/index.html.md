---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='mailto:development@eskesmedia.nl'>Send us an email</a>
  - <a href='https://www.eskesmedia.nl'>Copyright Eskes Media B.V.</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the TicketJames API. Use our API to register barcode scanners and synchrosise scans.

The TicketJames API is a work in progress and currently under heavy development.

Currently, the only function of this API is to register mobile phones (Android, iOS) as barcode scanners. In future releases, API functionality will probably be expanded with:

- activity callbacks
- coupon check
- barcode check


# Localization
TicketJames.com supports multiple languages:

- English (en)
- Dutch (nl) __(default)__
- Germen (de)
- French (fr)

The outputted format in the API is localized, when the locale parameter is present in the URI: 

- `https://api.ticketjames.com/nl/api/<endpoint>`
- `https://api.ticketjames.com/en/api/<endpoint>`
- `https://api.ticketjames.com/de/api/<endpoint>`

<aside class="notice">
Status messages will always be in English. When information is unavailable in the requested language, the projects default locale will be used.
</aside>


# Authentication
The TicketJames API doesn't required an API key or login credentials at this point. Only registered and accepted devises can communicate with the API.

# Barcode Scanners

## Register a new barcode scanner

```shell
curl "https://api.ticketjames.com/nl/api/barcode_scanners"
  -X POST
  -d '{"devise_uid":"65888ba4-9570-43e7-b701-71cb3f4d2549", "user_agent":"Mozilla/5.0 (iPhone; CPU iPhone OS 9_1 like Mac OS X) AppleWebKit/601.1.46 (KHTML, like Gecko) Version/9.0 Mobile/13B143"}' 
  -H "Content-Type: application/json"
```

> The above command returns a `HTTP 200` status and JSON structured like this:

```json
  {
    "status": "success"
  }

```

This endpoint registers a new devise. After registration, the admin user will see this devise appear (in real time) in the 'barcode scanners' option in the backoffice. He has the option to rename the devise or block permissions.

### HTTP Request

`POST https://api.ticketjames.com/nl/api/barcode_scanners`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
devise_uid | true | This is the primary key for the devise in TicketJames' backoffice.
user_agent | false | If given, the user agent will be saved in TicketJames and linked to this devise. It will be used for pretty naming the devise when no name is given. E.g: 'iPhone 9' instead of 'unkwown device'

<!-- <aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>
 -->
## Get a list of projects for a specific Barcode Scanner

```shell
curl "https://api.ticketjames.com/en/api/barcode-scanners/65888ba4-9570-43e7-b701-71cb3f4d2549/projects"
  -H "Content-Type: application/json"
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 2,
    "title": "Demo Project",
    "events": [
      {
        "id": 456,
        "starts_at": "2018-06-19 20:00:00 +0200",
        "name": "Opening night",
        "total_tickets": 245,
        "total_scanned": 42,
        "scannable_from": "2018-06-19 13:00:00 +0200",
        "scannable_until": "2018-06-19 23:00:00 +0200"
      },
      {
        "id": 457,
        "starts_at": "2018-06-20 20:00:00 +0200",
        "name": "VIP night",
        "total_tickets": 212,
        "total_scanned": 0,
        "scannable_from": "2018-06-20 13:00:00 +0200",
        "scannable_until": "2018-06-20 23:00:00 +0200"
      }
    ]
  }
]
```

This endpoint retrieves a list of projects and events.

### HTTP Request

`GET https://api.ticketjames.com/<LOCALE>/api/barcode-scanners/<DEVICE-UID>/projects`

### URL Parameters

Parameter | Description
--------- | -----------
UID | The UID of an already registered devise.

<aside class="warning">
When an barcode scanner is not registered, the server will return a <code>http 404 not found</code> message.
</aside>


## Get a list of scannable barcodes for a specific Barcode Scanner

```shell
curl "https://api.ticketjames.com/en/api/barcode-scanners/65888ba4-9570-43e7-b701-71cb3f4d2549/tickets?since=1529412079"
-H "Content-Type: application/json"
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 2564,
    "event_id": 456,
    "scannable_from": "2018-06-19 13:00:00 +0200",
    "scannable_until": "2018-06-19 23:00:00 +0200",
    "barcode": "291211",
    "title": "Opening Night - 1 Adult",
    "description": "Row 3, seat 12",
    "status": "valid"
  },
  {
    "id": 2565,
    "event_id": 456,
    "scannable_from": "2018-06-19 13:00:00 +0200",
    "scannable_until": "2018-06-19 23:00:00 +0200",
    "barcode": "034255",
    "title": "Opening Night - 1 Child",
    "description": "Row 7, seat 8",
    "status": "scanned"
  },
  {
    "id": 3216,
    "event_id": 457,
    "scannable_from": "2018-06-20 13:00:00 +0200",
    "scannable_until": "2018-06-20 23:00:00 +0200",
    "barcode": "655779",
    "title": "Vip Night - 1 Adult",
    "description": "Wheelchair - Row 2, seat 12",
    "status": "canceled"
  }
]
```

This endpoint requests a list of known tickets that can be scanned by the given barcode scanner.

Use this request every time you want to synchronise tickets from the server to your mobile device (barcode scanner). When this endpoint is called without the `since` parameter, the response JSON will containt __all available__ tickets. This can potentially be a very, very large list, e.gl: thousands of records.

Therefore, you will probably only want to call this endpoint just the first time. When 

### HTTP Request

`GET https://api.ticketjames.com/<LOCALE>/api/barcode-scanners/<DEVICE-UID>/tickets?since=<UNIX-TIMESTAMP>`

### URL Parameters

Parameter | Required | Description
--------- | -------- | -----------
UID | true | The UID of an already registered devise.
since | false | A timestamp in <a target="_blank" href="https://www.unixtimestamp.com/">Unix format</a> (seconds since epoch). 

