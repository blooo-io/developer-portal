---
title: Manifest
subtitle:
tags: [non dapp, live app, ledger live app]
category: Live Application
toc: true
layout: doc
---



To be able to test and integrate your application, you first need to write your application Manifest file.
This file must contain some mandatory information, such as the app package names, the components, the permissions needed, the hardware and software features, etc.

Example of Manifest (JSON format) for Moonpay application:

```json
{
    "id": "moonpay",
    "name": "Moonpay",
    "url": "https://buy.moonpay.com",
    "homepageUrl": "https://www.moonpay.com",
    "icon": "https://cdn.live.ledger.com/icons/platform/moonpay.png",
    "platform": "all",
    "apiVersion": "0.0.1",
    "manifestVersion": "1",
    "branch": "stable",
    "categories": ["buy"],
    "currencies": [
      "bitcoin",
      "ethereum"
    ],
    "content": {
      "shortDescription": {
        "en": "A fast, simple, and secure way to buy crypto."
      },
      "description": {
        "en": "MoonPay accepts all major payment methods and over 30 fiat currencies, enabling people from all over the world to buy crypto."
      }
    },
    "permissions": [],
    "domains": [
      "https://*"
    ]
  }
```

Here is the list of the mandatory fields required in your Manifest file:

- `id`: the identification of your application. Must be in lowercase.
Type: string.
- `name`: the name of your application ("Moonpay" in this example).
Type: string.
- `url`: the URL of your web application.
Type: string.
- `homepageUrl`: the homepage of your service, for instance "https://www.google.fr/"
Type: string.
- `icon`: a link to the icon being displayed in the Ledger Live Discover section. Will be hosted on Ledger CDN before being released in production.
Type: url.
- `platform`: a parameter to determine on which platform (desktop, mobile, iOS, android) your service will be available. By default you should set the value to "all".
Type: string.
- `apiVersion`: the API version, by default "0.0.1".
Type: string.
- `manifestVersion`: the manifest version, by default should be "1".
Type: string.
- `branch`: the specific branch used by Ledger to deploy the changes. Can take the values stable | experimental | debug | soon. By default you should set it to  "stable". The value “soon” will mark your app as “Coming soon” and it won’t be usable.
Type: string.
- `categories`: a JSON array containing metadata information about your application. For instance : ["staking","defi" ]
Type: list(string).
- `currencies`: a JSON array describing the currency/network being used by your application. For instance ["ethereum",”polygon”]
Type: list(string).
- `content`: a description of your service. It wiill be visible in the entry card of your application.
Type: l18n strings.
- `permissions`:
- `domains`:

(TBD) Manifest validation tool: your Manifest will be automatically rejected by Ledger Live if its invalid.
