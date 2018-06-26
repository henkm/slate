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
  - changelog
  - barcode_scanner

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
After a successful registration, the API returns a `key`. This key is needed for all communication. Provide the `key` as a header in each request.

