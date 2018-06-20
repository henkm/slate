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

Welcome to the TicketJames API. Use our API to register barcode scanners and synchronize scans.

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

- `https://api.ticketjames.com/nl/api/v1/<endpoint>`
- `https://api.ticketjames.com/en/api/v1/<endpoint>`
- `https://api.ticketjames.com/de/api/v1/<endpoint>`

<aside class="notice">
Status messages will always be in English. When information is unavailable in the requested language, the projects default locale will be used.
</aside>


# Authentication
The TicketJames API doesn't required an API key or login credentials at this point. Only registered and accepted devises can communicate with the API.

# Barcode Scanners

## Register a new barcode scanner

```shell
curl "https://api.ticketjames.com/nl/api/v1/barcode_scanners"
  -X POST
  -d '{"devise_uid":"65888ba4-9570-43e7-b701-71cb3f4d2549", "user_agent":"Mozilla/5.0 (iPhone; CPU iPhone OS 9_1 like Mac OS X) AppleWebKit/601.1.46 (KHTML, like Gecko) Version/9.0 Mobile/13B143", "type":"project", id":"2", "key":"0e3e0fee239d849b82b5db788ddcb8e3"}' 
  -H "Content-Type: application/json"
```

> The above command returns a `HTTP 200` status and JSON structured like this:

```json
  {
    "status": "success",
    "message": "Congratulations, you now have access to 'Demo Project', 432 tickets will be imported."
  }

```

This endpoint registers a new devise. After registration, the admin user will see this devise appear (in real time) in the 'barcode scanners' option in the backoffice. He has the option to rename the devise or block permissions.

### HTTP Request

`POST https://api.ticketjames.com/nl/api/v1/barcode_scanners`


### Obtain parameters from QR code
How does devise registration work? The project owner invites an employee to scan the tickets for his event(s). The project owner prints/shows a special QR code from the TicketJames backoffice. By scanning this code, the employees phone is now registered to the designated event(s). The secret is in the QR code. When scanned, the QR code will read something like this:

`eyJ0eXBlIjoicHJvamVjdCIsICJpZCI6Miwia2V5IjoiMGUzZTBmZWUyMzlkODQ5YjgyYjVkYjc4OGRkY2I4ZTMifQ==`

Your software should at this point think: _hey, this looks like a Base64 encrypted string, I'll decode it!_.

The string above, decodes to:

`{"type":"project", "id":2,"key":"0e3e0fee239d849b82b5db788ddcb8e3"}`

The decoded parameters can then be used to register the devise with a POST to the `barcode_scanners` endpoint.

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
devise_uid | true | This is the primary key for the devise in TicketJames' backoffice.
user_agent | false | If given, the user agent will be saved in TicketJames and linked to this devise. It will be used for pretty naming the devise when no name is given. E.g: 'iPhone 9' instead of 'unkwown device'. Please provide the full user agent string, it will be parsed on the server.
type | true | The type (one of ['company','project','event','group','location']) obtained from the QR code
id | true | The ID obtained from the QR code
key | true | The `key` object from the QR code

<!-- <aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>
 -->
## Get a list of projects for a specific Barcode Scanner

```shell
curl "https://api.ticketjames.com/en/api/v1/barcode-scanners/65888ba4-9570-43e7-b701-71cb3f4d2549/projects"
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
        "is_blocked": false,
        "scannable_from": "2018-06-19 13:00:00 +0200",
        "scannable_until": "2018-06-19 23:00:00 +0200"
      },
      {
        "id": 457,
        "starts_at": "2018-06-20 20:00:00 +0200",
        "name": "VIP night",
        "total_tickets": 212,
        "total_scanned": 0,
        "is_blocked": false,
        "scannable_from": "2018-06-20 13:00:00 +0200",
        "scannable_until": "2018-06-20 23:00:00 +0200"
      }
    ]
  }
]
```

This endpoint retrieves a list of projects and events.

### HTTP Request

`GET https://api.ticketjames.com/<LOCALE>/api/v1/barcode-scanners/<DEVICE-UID>/projects`

### URL Parameters

Parameter | Description
--------- | -----------
UID | The UID of an already registered devise.

<aside class="warning">
When a barcode scanner is not registered, the server will return a <code>http 404 not found</code> message.
</aside>


### Return message
The JSON response contains a list of `projects`, each containing one ore more `events`.

Attribute | Description
--------- | -----------
id | Unique event ID
starts_at | Start time of the event
name | (localized) name of the event
total_tickets | The total number of valid tickets for this event
total_scanned | The total number of tickets already scanned
is_blocked | Returns `false` by default, but can be `true` if the scanning device is blocked by the project owner.
scannable_from | Tickets can be scanned later than given time
scannable_from | Tickets can be scanned until given time



## Get a list of scannable barcodes for a specific Barcode Scanner

```shell
curl "https://api.ticketjames.com/en/api/v1/barcode-scanners/65888ba4-9570-43e7-b701-71cb3f4d2549/tickets?since=1529412079"
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

Use this request every time you want to synchronize tickets from the server to your mobile device (barcode scanner). When this endpoint is called without the `since` parameter, the response JSON will containt __all available__ tickets. This can potentially be a very, very large list, e.gl: thousands of records.

Therefore, you will probably only want to call this endpoint just the first time. When you call the endpoint __with__ the `since` parameter, the resulting JSON will only contain tickets that have been changed since the given time:
 - New tickets
 - Tickets with changed details (seat number, title, etc.)
 - Tickets with changed status (valid -> scanned, or valid -> canceled)

### HTTP Request

`GET https://api.ticketjames.com/<LOCALE>/api/v1/barcode-scanners/<DEVICE-UID>/tickets?since=<UNIX-TIMESTAMP>`

### URL Parameters

Parameter | Required | Description
--------- | -------- | -----------
locale | false | The requested locale for titles and descriptions.
UID | true | The UID of an already registered devise.
since | false | A timestamp in <a target="_blank" href="https://www.unixtimestamp.com/">Unix format</a> (seconds since epoch). 

<aside class="notice">
Please note that the 'barcode' attribute is of type String, even though it is always a number. The reason for this, is that the first digit of a barcode can be a '0'. Storing these numbers as Integer will result in unexpected errors.
</aside>


## Submit local scan actions

```shell
curl "https://api.ticketjames.com/en/api/v1/barcode-scanners/65888ba4-9570-43e7-b701-71cb3f4d2549/scan"
-X POST
-d '{"barcodes":["123456", "789012"]' 
-H "Content-Type: application/json"
```

> The above command returns JSON structured like this:

```json
[
  {
    "123456": "scanned",
    "ticket":{
      "id": 2565,
      "event_id": 456,
      "scannable_from": "2018-06-19 13:00:00 +0200",
      "scannable_until": "2018-06-19 23:00:00 +0200",
      "barcode": "123456",
      "title": "Opening Night - 1 Child",
      "description": "Row 4, seat 8",
      "status": "scanned"
    }
  },
  {
    "789012": "invalid"
  },
  {
    "655779": "invalid_time",
    "ticket": {
      "id": 2565,
      "event_id": 456,
      "scannable_from": "2018-06-20 13:00:00 +0200",
      "scannable_until": "2018-06-20 23:00:00 +0200",
      "barcode": "034255",
      "title": "Opening Night - 1 Child",
      "description": "Row 7, seat 8",
      "status": "valid"
    }
  }
]
```

Use this endpoint to send a batch of barocdes to the API. Both valid and invalid barcodes can be sent. The API will generate and return a response for each barcode and return an JSON Array containing all the responses.

<aside class="notice">
Even if you are making a 'scan' request for every ticket scanned, the `barcodes` parameter should be an Array (of 1 object, in that case).
</aside>


### Suggested workflow
The suggested workflow is to sync __asynchronously__ and to sync __often__. Every time you scan a ticket locally, just check it with your local database. You should be able to assume it is up to date (you are the one who should make sure of that).

### What if a ticket is scanned on the device, but not found in the database?
This would be a good time to make a call to this endpoint and check the status for this barcode. Maybe it has just been purchased and your database isn't up to date yet. 

### HTTP Request

`GET https://api.ticketjames.com/<LOCALE>/api/v1/barcode-scanners/<DEVICE-UID>/scan`

### URL Parameters

Parameter | Required | Description
--------- | -------- | -----------
locale | false | 
UID | true | The UID of an already registered devise.

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
barcodes | true | An array of barcodes to sync back to the server.

### Possible status messages

Message | Description
------- | -----------
success | The ticket will be marked as scanned. OK!
already_scanned | This ticket has already been scanned.
canceled | This ticket used to be valid, but has been canceled.
invalid | No tickets found with this barcode. Is your device registered for this project?
invalid_time | This tickets is valid, but the scan time isn't within the limits of 'scannable_from' and 'scannable_until'. The ticket **will not be marked as scanned**.