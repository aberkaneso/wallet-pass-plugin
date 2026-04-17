# Apple Wallet Technical Reference

Complete technical documentation for creating Apple Wallet passes (`.pkpass` files). This reference covers all pass types, fields, images, and distribution methods.

## Table of Contents

1. [pass.json Required Fields](#1-passjson-required-fields)
2. [Visual Appearance Fields](#2-visual-appearance-fields)
3. [Pass Types and Field Groups](#3-pass-types-and-field-groups)
4. [PassFieldContent Object](#4-passfieldcontent-object)
5. [Barcodes](#5-barcodes)
6. [Relevance](#6-relevance)
7. [Expiration and Voiding](#7-expiration-and-voiding)
8. [Grouping](#8-grouping)
9. [Associated Apps](#9-associated-apps)
10. [NFC](#10-nfc)
11. [Semantic Tags](#11-semantic-tags)
12. [Image Specifications](#12-image-specifications)
13. [Bundle Structure](#13-bundle-structure)
14. [Personalization](#14-personalization)
15. [Localization](#15-localization)
16. [Distribution](#16-distribution)

---

## 1. pass.json Required Fields

Every Apple Wallet pass must include these required fields at the top level of `pass.json`:

### Core Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `formatVersion` | Integer | Always `1`. Do not use other values. |
| `passTypeIdentifier` | String | Reverse-domain format matching your signing certificate (e.g., `pass.com.example.myapp`). Must be registered in Apple Developer portal. |
| `serialNumber` | String | Unique identifier within each `passTypeIdentifier`. No two passes with the same `passTypeIdentifier` can have identical `serialNumber` values. |
| `teamIdentifier` | String | Your Apple Developer Team ID (10-character identifier from your Apple account). |
| `organizationName` | String (Localizable) | Name of organization issuing the pass. Displayed to users. Can be localized via `pass.strings`. |
| `description` | String (Localizable) | Human-readable description of pass purpose. Used for Siri, VoiceOver, and accessibility. Can be localized. |

### Example Minimal pass.json

```json
{
  "formatVersion": 1,
  "passTypeIdentifier": "pass.com.example.myapp",
  "serialNumber": "E5982H-I2",
  "teamIdentifier": "A1B2C3D4E5",
  "organizationName": "Acme Corporation",
  "description": "Acme Store Card"
}
```

---

## 2. Visual Appearance Fields

Control the visual presentation of the pass across all Apple platforms.

### Color Fields

All colors use RGB format: `rgb(r, g, b)` where r, g, b are integers 0–255.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `backgroundColor` | String | No | Background color of pass front. Default: white. Format: `rgb(255, 255, 255)` |
| `foregroundColor` | String | No | Text color on pass front. Default: black. Format: `rgb(0, 0, 0)` |
| `labelColor` | String | No | Label text color (field labels). Automatically set if not specified. Format: `rgb(128, 128, 128)` |

### Text and Appearance

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `logoText` | String (Localizable) | No | Short text displayed next to logo image. Typically 1–3 words. Can be localized. |
| `suppressStripShine` | Boolean | No | When `true`, disables the shiny reflection overlay on strip images. Default: `false`. |

### Event Ticket Fields (Poster Style)

Additional fields available **only** for `eventTicket` pass type, when using poster-style display:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `eventLogoText` | String (Localizable) | No | Supplemental text displayed with event logo on poster. Localizable. |
| `footerBackgroundColor` | String | No | Background color of footer area. Format: `rgb(r, g, b)`. |
| `suppressHeaderDarkening` | Boolean | No | When `true`, prevents automatic darkening of header on poster. Default: `false`. |
| `useAutomaticColors` | Boolean | No | When `true`, Wallet automatically adjusts colors based on background image. Default: `false`. |

### Example Visual Configuration

```json
{
  "backgroundColor": "rgb(255, 255, 255)",
  "foregroundColor": "rgb(0, 0, 0)",
  "labelColor": "rgb(128, 128, 128)",
  "logoText": "ACME Store",
  "suppressStripShine": false
}
```

---

## 3. Pass Types and Field Groups

Apple Wallet supports **exactly five pass types**. Each pass.json must specify **exactly one** pass type. Field groups control how data is displayed on the pass.

### Pass Type Overview

| Type | Key | Best For | Special Features |
|------|-----|----------|------------------|
| Boarding Pass | `boardingPass` | Flight, train, bus, boat tickets | Transit type required, dedicated footer, URL actions |
| Coupon | `coupon` | Discounts, promotions | Compact format, strip image |
| Event Ticket | `eventTicket` | Sports, concerts, conferences | Poster display option, accessibility features, event semantics |
| Store Card | `storeCard` | Loyalty, membership, gift cards | Compact format, strip image |
| Generic | `generic` | Any other use case | Most flexible, background image support |

### Field Groups

All pass types support these field group arrays. Each array contains `PassFieldContent` objects (see Section 4).

| Field Group | Required | Description |
|------------|----------|-------------|
| `headerFields` | No | Small-text fields above primary fields. Row 1 of information. |
| `primaryFields` | No | Main content. Most prominent. Limited count per pass type. |
| `secondaryFields` | No | Supporting information. Row 2. Limited count per pass type. |
| `auxiliaryFields` | No | Additional details. Row 3+. Limited count per pass type. |
| `backFields` | No | Information on reverse side of pass. Unlimited count. |
| `additionalInfoFields` | No | **Event Tickets Only (Poster)**: Additional rows of information below main fields. |

### Field Limits by Pass Type

| Pass Type | Header | Primary | Secondary | Auxiliary | Back |
|-----------|--------|---------|-----------|-----------|------|
| `boardingPass` | 2 | 2 | 5 | 5 | Unlimited |
| `coupon` | 1 | 1 | 4 | 4 | Unlimited |
| `eventTicket` | 3 | 2 (1 poster) | 4 | 4 | Unlimited |
| `storeCard` | 3 | 1 | 4 | 4 | Unlimited |
| `generic` | 3 | 1 | 4 | 4 | Unlimited |

### Boarding Pass Example

```json
{
  "boardingPass": {
    "transitType": "PKTransitTypeAir",
    "headerFields": [
      {
        "key": "airline",
        "label": "Airline",
        "value": "Acme Air"
      }
    ],
    "primaryFields": [
      {
        "key": "gate",
        "label": "Gate",
        "value": "42"
      }
    ],
    "secondaryFields": [
      {
        "key": "departure",
        "label": "Departs",
        "value": "2:15 PM"
      }
    ],
    "auxiliaryFields": [
      {
        "key": "seat",
        "label": "Seat",
        "value": "42A"
      }
    ]
  }
}
```

### Boarding Pass Transit Types

| Value | Transport |
|-------|-----------|
| `PKTransitTypeAir` | Airplane |
| `PKTransitTypeTrain` | Train |
| `PKTransitTypeBus` | Bus |
| `PKTransitTypeBoat` | Boat/Ferry |
| `PKTransitTypeGeneric` | Other transit |

### Boarding Pass URL Actions

Available only in `boardingPass` type. Add these at the top level alongside `boardingPass` object:

| Field | Type | Description |
|-------|------|-------------|
| `changeSeatURL` | String | URL to change seat selection |
| `entertainmentURL` | String | In-flight entertainment system |
| `purchaseAdditionalBaggageURL` | String | Buy extra baggage |
| `purchaseLoungeAccessURL` | String | Airport lounge access |
| `purchaseWifiURL` | String | In-flight WiFi purchase |
| `upgradeURL` | String | Cabin upgrade options |
| `managementURL` | String | General trip management |
| `registerServiceAnimalURL` | String | Service animal registration |
| `reportLostBagURL` | String | Lost baggage claim |
| `requestWheelchairURL` | String | Accessibility services |
| `trackBagsURL` | String | Baggage tracking |
| `transitProviderEmail` | String | Support email |
| `transitProviderPhoneNumber` | String | Support phone |
| `transitProviderWebsiteURL` | String | Airline/provider website |

### Event Ticket URL Actions

Available only in `eventTicket` type:

| Field | Type | Description |
|-------|------|-------------|
| `accessibilityURL` | String | Accessibility information |
| `addOnURL` | String | Merchandise or upsells |
| `bagPolicyURL` | String | Venue bag policy |
| `contactVenueEmail` | String | Venue contact email |
| `contactVenuePhone` | String | Venue phone number |
| `contactVenueWebsite` | String | Venue website |
| `directionsInformationURL` | String | Directions/maps |
| `merchandiseURL` | String | Event merchandise store |
| `orderFoodURL` | String | Food ordering |
| `parkingInformationURL` | String | Parking details |
| `purchaseParkingURL` | String | Purchase parking |
| `sellURL` | String | Resale/transfer URL |
| `transferURL` | String | Transfer ticket |
| `transitInformationURL` | String | Public transportation info |

### Coupon and Store Card

```json
{
  "coupon": {
    "primaryFields": [
      {
        "key": "discount",
        "label": "Discount",
        "value": "30% Off"
      }
    ],
    "secondaryFields": [
      {
        "key": "expiry",
        "label": "Expires",
        "value": "December 31, 2025"
      }
    ]
  }
}
```

### Generic Pass

```json
{
  "generic": {
    "primaryFields": [
      {
        "key": "custom",
        "label": "Information",
        "value": "Pass-specific data"
      }
    ]
  }
}
```

---

## 4. PassFieldContent Object

Every field in field groups is a `PassFieldContent` object. This object controls how a piece of information appears on the pass.

### Required Properties

| Property | Type | Description |
|----------|------|-------------|
| `key` | String | Unique identifier within the pass. Used to update fields via web service. No spaces or special characters recommended. |
| `value` | String, Number, or ISO 8601 Date | The data displayed. Can be text, number, or date (e.g., `"2025-12-31T23:59:59-08:00"`). |

### Optional Display Properties

| Property | Type | Valid Values | Description |
|----------|------|--------------|-------------|
| `label` | String (Localizable) | Any text | Text label shown above/beside value. Localizable. |
| `attributedValue` | String | HTML with only `<a href>` tags | Rich text value (with links). Overrides `value` for display. **Not on watchOS**. Only `<a href>` tags supported. |
| `changeMessage` | String (Localizable) | Format string with `%@` | Alert text shown when field updates. Must contain `%@` placeholder. Example: `"Updated to %@"`. |
| `textAlignment` | String | `PKTextAlignmentLeft`, `PKTextAlignmentCenter`, `PKTextAlignmentRight`, `PKTextAlignmentNatural` | Text alignment. Default: `PKTextAlignmentNatural`. **Not valid for primary or back fields**. |

### Number and Currency Formatting

| Property | Type | Valid Values | Description |
|----------|------|--------------|-------------|
| `currencyCode` | String | ISO 4217 (e.g., `"USD"`, `"EUR"`) | Used when `value` is monetary amount. Applies automatic currency formatting and symbol. |
| `numberStyle` | String | `PKNumberStyleDecimal`, `PKNumberStylePercent`, `PKNumberStyleScientific`, `PKNumberStyleSpellOut` | Formatting style for numeric values. |

### Date and Time Formatting

| Property | Type | Valid Values | Description |
|----------|------|--------------|-------------|
| `dateStyle` | String | `PKDateStyleNone`, `PKDateStyleShort`, `PKDateStyleMedium`, `PKDateStyleLong`, `PKDateStyleFull` | How to display date portion. |
| `timeStyle` | String | Same as `dateStyle` | How to display time portion. |
| `ignoresTimeZone` | Boolean | `true`, `false` | Default: `false`. If `true`, time is displayed without timezone conversion. |
| `isRelative` | Boolean | `true`, `false` | Default: `false`. If `true`, date shown as relative (e.g., "2 days away"). |

### Field Grouping and Layout

| Property | Type | Description |
|----------|------|-------------|
| `row` | Integer | 0-based row index for `auxiliaryFields` and `secondaryFields`. Controls vertical placement. Fields with same `row` appear horizontally. |

### Data Detection (Back Fields Only)

| Property | Type | Valid Values | Description |
|----------|------|--------------|-------------|
| `dataDetectorTypes` | Array | `PKDataDetectorTypePhoneNumber`, `PKDataDetectorTypeLink`, `PKDataDetectorTypeAddress`, `PKDataDetectorTypeCalendarEvent` | Back fields only. Enables auto-detection and linking. Default: all enabled. Empty array `[]` disables all. |

### Per-Field Semantic Tags

| Property | Type | Description |
|----------|------|-------------|
| `semantics` | Object | Per-field semantic tag overrides (see Section 11). Overrides top-level semantics for this field. |

### Complete Field Examples

#### Simple Text Field
```json
{
  "key": "description",
  "label": "Details",
  "value": "VIP Access Pass"
}
```

#### Currency Field
```json
{
  "key": "balance",
  "label": "Balance",
  "value": "150.50",
  "currencyCode": "USD",
  "numberStyle": "PKNumberStyleDecimal"
}
```

#### Date Field
```json
{
  "key": "eventDate",
  "label": "Event",
  "value": "2025-12-25T19:00:00-08:00",
  "dateStyle": "PKDateStyleLong",
  "timeStyle": "PKDateStyleShort",
  "ignoresTimeZone": false
}
```

#### Field with Change Alert
```json
{
  "key": "status",
  "label": "Status",
  "value": "Boarding Now",
  "changeMessage": "Status updated to %@"
}
```

#### Rich Text Field (with link)
```json
{
  "key": "details",
  "label": "More Info",
  "value": "https://example.com",
  "attributedValue": "<a href=\"https://example.com\">View Full Details</a>"
}
```

#### Percentage Field
```json
{
  "key": "discount",
  "label": "Discount",
  "value": "0.30",
  "numberStyle": "PKNumberStylePercent"
}
```

#### Text Field with Alignment
```json
{
  "key": "note",
  "label": "Special Instructions",
  "value": "Hand over to gate agent",
  "textAlignment": "PKTextAlignmentCenter"
}
```

#### Back Field with Data Detection
```json
{
  "key": "contact",
  "label": "Support",
  "value": "Call 1-800-ACME or visit acme.com",
  "dataDetectorTypes": ["PKDataDetectorTypePhoneNumber", "PKDataDetectorTypeLink"]
}
```

#### Field with Row Placement
```json
{
  "key": "departure",
  "label": "Departs",
  "value": "2:15 PM",
  "row": 0
}
```

---

## 5. Barcodes

Passes can include barcodes for scanning at point of sale, entry points, or for data transmission. Use the newer `barcodes` array (preferred) or the deprecated `barcode` key.

### Barcodes Array (Preferred)

Use an array of barcode objects. System will use the first barcode format the device can display.

```json
{
  "barcodes": [
    {
      "format": "PKBarcodeFormatQR",
      "message": "123456789012",
      "messageEncoding": "iso-8859-1",
      "altText": "Barcode: 123456789012"
    }
  ]
}
```

### Barcode Object Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `format` | String | Yes | Barcode format (see table below). |
| `message` | String | Yes | The data encoded in the barcode. Max length varies by format. |
| `messageEncoding` | String | Yes | Character encoding. Typically `"iso-8859-1"` or `"utf-8"`. |
| `altText` | String | No | Fallback text displayed if barcode cannot be scanned. Also used for accessibility. |

### Barcode Formats

| Format | Description | Supported On | Max Length |
|--------|-------------|--------------|-----------|
| `PKBarcodeFormatQR` | QR Code 2D barcode | iOS, watchOS | ~4,296 chars |
| `PKBarcodeFormatPDF417` | PDF417 2D barcode | iOS | ~1,115 chars |
| `PKBarcodeFormatAztec` | Aztec 2D barcode | iOS | ~3,832 chars |
| `PKBarcodeFormatCode128` | Code 128 1D barcode | iOS only (not watchOS) | ~80 chars |

### Multiple Barcodes Example

```json
{
  "barcodes": [
    {
      "format": "PKBarcodeFormatQR",
      "message": "https://example.com/pass/ABC123",
      "messageEncoding": "iso-8859-1",
      "altText": "Scan for details"
    },
    {
      "format": "PKBarcodeFormatCode128",
      "message": "ABC123456789",
      "messageEncoding": "iso-8859-1",
      "altText": "Code: ABC123456789"
    }
  ]
}
```

### Deprecated Barcode Field

The singular `barcode` key is deprecated but still supported:

```json
{
  "barcode": {
    "format": "PKBarcodeFormatQR",
    "message": "123456789012",
    "messageEncoding": "iso-8859-1"
  }
}
```

---

## 6. Relevance

Control when and where the pass should be prominent in Wallet based on location, time, and Bluetooth proximity.

### Location-Based Relevance

Display pass when user is near specific locations:

```json
{
  "locations": [
    {
      "latitude": 37.7749,
      "longitude": -122.4194,
      "relevantText": "You're near our San Francisco store!"
    },
    {
      "latitude": 40.7128,
      "longitude": -74.0060,
      "altitude": 10,
      "relevantText": "Welcome to New York!"
    }
  ],
  "maxDistance": 500
}
```

**Locations Array Properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `latitude` | Number | Yes | Latitude in degrees. Range: -90 to 90. |
| `longitude` | Number | Yes | Longitude in degrees. Range: -180 to 180. |
| `altitude` | Number | No | Altitude in meters. Used for enhanced precision. |
| `relevantText` | String (Localizable) | No | Text displayed when user enters location. Localizable. |

**Location Array Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `locations` | Array | Up to **10 location objects**. Additional locations ignored. |
| `maxDistance` | Integer | Maximum distance in meters from location where pass is relevant. Default: no limit. Example: `500` for 500 meters. |

### Time-Based Relevance

Use `relevantDates` array (preferred) or deprecated `relevantDate`:

```json
{
  "relevantDates": [
    {
      "startDate": "2025-12-24T09:00:00-08:00",
      "endDate": "2025-12-25T23:59:59-08:00"
    },
    {
      "startDate": "2025-12-31T19:00:00-08:00",
      "endDate": "2026-01-01T02:00:00-08:00"
    }
  ]
}
```

**Relevant Dates Properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `startDate` | String (ISO 8601) | Yes | When pass becomes relevant. Includes timezone (e.g., `2025-12-31T09:00:00-05:00`). |
| `endDate` | String (ISO 8601) | Yes | When pass is no longer relevant. Includes timezone. |

### Beacon-Based Relevance

Trigger pass display when near Bluetooth Low Energy (BLE) beacons:

```json
{
  "beacons": [
    {
      "proximityUUID": "72C4DB47-00FF-41B8-8E32-B146DD0BC676",
      "relevantText": "Welcome! Check in here."
    },
    {
      "proximityUUID": "E2C56DB5-DFFB-48D2-B060-D0F5A71096E0",
      "major": 1,
      "minor": 1,
      "relevantText": "Premium seating area"
    }
  ]
}
```

**Beacons Array Properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `proximityUUID` | String (UUID) | Yes | UUID of the beacon. Format: `XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX`. |
| `major` | Integer | No | Major beacon identifier (0–65535). Used for grouping. |
| `minor` | Integer | No | Minor beacon identifier (0–65535). Used for individual beacons. |
| `relevantText` | String (Localizable) | No | Text shown when user is in beacon proximity. Localizable. |

### Deprecated Relevant Date (Singular)

```json
{
  "relevantDate": "2025-12-31T23:59:59-08:00"
}
```

---

## 7. Expiration and Voiding

### Expiration Date

Mark a pass as no longer valid at a specific time:

```json
{
  "expirationDate": "2026-12-31T23:59:59-08:00"
}
```

| Property | Type | Description |
|----------|------|-------------|
| `expirationDate` | String (ISO 8601) | Date and time when pass expires. Must include timezone. Expired passes appear grayed out in Wallet. |

### Voiding

Mark a pass as used or redeemed:

```json
{
  "voided": true
}
```

| Property | Type | Description |
|----------|------|-------------|
| `voided` | Boolean | When `true`, pass appears grayed out in Wallet, indicating it's been used. Cannot be changed back via web service (must issue new pass). |

---

## 8. Grouping

Group related passes together in Wallet. All passes with the same `passTypeIdentifier`, `groupingIdentifier`, and pass type appear as a single stack.

```json
{
  "groupingIdentifier": "boarding-pass-2025-flight-123",
  "passTypeIdentifier": "pass.com.example.airline"
}
```

| Property | Type | Description |
|----------|------|-------------|
| `groupingIdentifier` | String | Identifier for grouping. Same type of pass with same ID groups together. |

**Grouping Logic:**
- Two passes group together if: `passTypeIdentifier` + `groupingIdentifier` + pass type (e.g., `boardingPass`) are **identical**.
- Each unique combination creates a separate group.
- Used for boarding passes (multiple flights), event tickets (multiple shows), etc.

---

## 9. Associated Apps

Link the pass to an iOS app in App Store. Users can seamlessly transition from Wallet to your app.

```json
{
  "associatedStoreIdentifiers": [123456789, 987654321],
  "appLaunchURL": "myapp://pass/ABC123"
}
```

| Property | Type | Description |
|----------|------|-------------|
| `associatedStoreIdentifiers` | Array | App Store IDs as integers. System uses first installed app. |
| `appLaunchURL` | String | Custom URL scheme to launch app with pass context (e.g., `myapp://pass/ID`). Handler must be in app's Info.plist. |

---

## 10. NFC

Embed NFC (Near Field Communication) data for tap-to-interact functionality (iPhone XS and later, Apple Watch Series 5+).

```json
{
  "nfc": [
    {
      "message": "https://example.com/pass/ABC123",
      "encryptionPublicKey": "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDk..."
    }
  ]
}
```

| Property | Type | Description |
|----------|------|-------------|
| `nfc` | Array | Array of NFC objects. Typically one per pass. |
| `message` | String | Data to transmit via NFC. Max 64 bytes. URL, serial number, or custom data. |
| `encryptionPublicKey` | String | Optional base64-encoded RSA public key for encrypting sensitive data. Allows pass updater to decrypt at scan time. |

---

## 11. Semantic Tags

Semantic tags provide machine-readable context to Apple Wallet, enabling:
- Siri integration and voice commands
- Automatic rich notifications
- Accessibility features
- Device lock screen display
- Smart suggestions

Add semantics at the **top level** of pass.json, or **per-field** in PassFieldContent objects.

### Top-Level Semantics Structure

```json
{
  "semantics": {
    "eventName": "Acme Concert 2025",
    "eventType": "PKEventTypeLivePerformance",
    "eventStartDate": "2025-12-31T20:00:00-08:00",
    "eventEndDate": "2025-12-31T23:59:59-08:00",
    "venueName": "Acme Hall",
    "venueLocation": {
      "latitude": 37.7749,
      "longitude": -122.4194
    }
  }
}
```

### General Semantic Tags

| Field | Type | Description |
|--------|------|-------------|
| `totalPrice` | Object | { `amount`: number, `currencyCode`: "USD" } – Total pass value. |
| `duration` | Integer | Duration in seconds. |
| `seats` | Array | Array of seat objects. See seat structure below. |
| `silenceRequested` | Boolean | If `true`, device silence mode recommended. |
| `membershipProgramName` | String | Name of membership program (e.g., "Gold Circle"). |
| `membershipProgramNumber` | String | Member ID or account number. |
| `personalizedNickname` | String | User-facing nickname for pass. |

#### Seat Object Structure

```json
{
  "seats": [
    {
      "seatDescription": "Premium Orchestra",
      "seatIdentifier": "C-12",
      "seatNumber": "12",
      "seatRow": "C",
      "seatSection": "Orchestra",
      "seatType": "Premium"
    }
  ]
}
```

| Field | Type | Description |
|--------|------|-------------|
| `seatDescription` | String | Human-readable seat description. |
| `seatIdentifier` | String | Unique seat code. |
| `seatNumber` | String | Seat number within row. |
| `seatRow` | String | Row identifier (e.g., "A", "B", "C"). |
| `seatSection` | String | Section name (e.g., "Orchestra", "Balcony"). |
| `seatType` | String | Seat class (e.g., "Premium", "Standard"). |

### Event Semantic Tags

| Field | Type | Description |
|--------|------|-------------|
| `eventName` | String | Event title (e.g., "Acme Concert 2025"). |
| `eventType` | String | Event category (see enum below). |
| `eventStartDate` | String (ISO 8601) | Event start time with timezone. |
| `eventEndDate` | String (ISO 8601) | Event end time with timezone. |
| `venueName` | String | Venue or location name. |
| `venueRoom` | String | Specific room/hall within venue. |
| `venuePhoneNumber` | String | Venue contact phone. |
| `venueEntrance` | String | Entrance identifier (e.g., "Main", "Side A"). |
| `venueLocation` | Object | Venue coordinates: { `latitude`: number, `longitude`: number }. |
| `artistIDs` | Array | Array of artist identifier strings. |
| `performerNames` | Array | Array of performer name strings. |
| `genre` | String | Event genre (e.g., "Rock", "Comedy"). |
| `subGenre` | String | Subgenre (e.g., "Alternative Rock"). |

#### Event Type Enum

| Value | Category |
|-------|----------|
| `PKEventTypeGeneric` | Unspecified event |
| `PKEventTypeLivePerformance` | Concert, theater, performance |
| `PKEventTypeMovie` | Cinema screening |
| `PKEventTypeSports` | Sports competition |
| `PKEventTypeConference` | Conference or convention |
| `PKEventTypeConvention` | Convention or expo |
| `PKEventTypeWorkshop` | Workshop or class |
| `PKEventTypeSocialGathering` | Party, wedding, reunion |

### Sports Semantic Tags

| Field | Type | Description |
|--------|------|-------------|
| `sportName` | String | Sport name (e.g., "Basketball"). |
| `leagueName` | String | League name (e.g., "NBA"). |
| `leagueAbbreviation` | String | League abbreviation (e.g., "NBA"). |
| `homeTeamName` | String | Home team name. |
| `homeTeamAbbreviation` | String | Home team abbreviation (e.g., "GSW"). |
| `homeTeamLocation` | String | Home team city. |
| `awayTeamName` | String | Away team name. |
| `awayTeamAbbreviation` | String | Away team abbreviation. |
| `awayTeamLocation` | String | Away team city. |

### Transit and Boarding Semantic Tags

| Field | Type | Description |
|--------|------|-------------|
| `boardingGroup` | String | Boarding group (e.g., "Group A", "1"). |
| `boardingSequenceNumber` | String | Position in boarding sequence. |
| `confirmationNumber` | String | Booking/confirmation code. |
| `passengerName` | String | Passenger full name. |
| `priorityStatus` | String | Priority level (e.g., "First Class", "Elite"). |
| `transitProvider` | String | Airline, train, or bus operator name. |
| `transitStatus` | String | Current status (e.g., "On Time", "Delayed"). |
| `transitStatusReason` | String | Reason for status (e.g., "Weather", "Mechanical"). |
| `vehicleType` | String | Type of vehicle (e.g., "Boeing 737", "Amtrak"). |
| `vehicleName` | String | Vehicle name or number. |
| `vehicleNumber` | String | Flight number, train number, etc. |

#### Departure and Arrival Tags

Departure tags (prefix "departure"):

| Field | Type | Description |
|--------|------|-------------|
| `departureAirportCode` | String | IATA code (e.g., "SFO"). |
| `departureAirportName` | String | Full airport name. |
| `departureGate` | String | Gate number. |
| `departureTerminal` | String | Terminal identifier. |
| `departurePlatform` | String | Platform number (trains, buses). |
| `departureStationName` | String | Station name. |
| `departureLocation` | Object | { `latitude`: number, `longitude`: number }. |
| `currentDepartureDate` | String (ISO 8601) | Current departure time. |
| `originalDepartureDate` | String (ISO 8601) | Original scheduled departure. |

Arrival tags (prefix "arrival/destination"):

| Field | Type | Description |
|--------|------|-------------|
| `destinationAirportCode` | String | IATA code. |
| `destinationAirportName` | String | Full airport name. |
| `destinationGate` | String | Gate number. |
| `destinationTerminal` | String | Terminal identifier. |
| `destinationPlatform` | String | Platform number. |
| `destinationStationName` | String | Station name. |
| `destinationLocation` | Object | { `latitude`: number, `longitude`: number }. |
| `currentArrivalDate` | String (ISO 8601) | Current arrival time. |
| `originalArrivalDate` | String (ISO 8601) | Original scheduled arrival. |

#### Flight-Specific Tags

| Field | Type | Description |
|--------|------|-------------|
| `flightCode` | String | Airline code + flight number (e.g., "AC123"). |
| `flightNumber` | String | Flight number only (e.g., "123"). |
| `airlineCode` | String | Airline IATA code (e.g., "AC"). |

#### WiFi Access (Transit)

| Field | Type | Description |
|--------|------|-------------|
| `wifiAccess` | Array | Array of WiFi network objects for in-flight/transit. |

WiFi Network Object:

```json
{
  "wifiAccess": [
    {
      "ssid": "AcmeAir-WiFi",
      "password": "PASS123",
      "isHidden": false
    }
  ]
}
```

| Field | Type | Description |
|--------|------|-------------|
| `ssid` | String | Network name. |
| `password` | String | Network password. |
| `isHidden` | Boolean | If `true`, network is hidden. |

### Balance Semantic Tag

| Field | Type | Description |
|--------|------|-------------|
| `balance` | Object | Account balance object (see structure below). |

Balance Object Structure:

```json
{
  "balance": {
    "amount": 150.50,
    "currencyCode": "USD",
    "balanceRefreshDate": "2025-04-08T12:00:00-08:00",
    "balanceRefreshLabel": "Last updated"
  }
}
```

| Field | Type | Description |
|--------|------|-------------|
| `amount` | Number | Balance value. |
| `currencyCode` | String | ISO 4217 currency code. |
| `balanceRefreshDate` | String (ISO 8601) | When balance was last updated. |
| `balanceRefreshLabel` | String | Label for refresh time. |

### Complete Semantic Tags Example

```json
{
  "semantics": {
    "eventName": "Acme Music Festival 2025",
    "eventType": "PKEventTypeLivePerformance",
    "eventStartDate": "2025-12-31T18:00:00-08:00",
    "eventEndDate": "2025-12-31T23:59:59-08:00",
    "venueName": "Golden Gate Park",
    "venueLocation": {
      "latitude": 37.7694,
      "longitude": -122.4862
    },
    "seats": [
      {
        "seatSection": "General Admission",
        "seatRow": "A",
        "seatNumber": "42"
      }
    ],
    "totalPrice": {
      "amount": 125.00,
      "currencyCode": "USD"
    }
  }
}
```

---

## 12. Image Specifications

Apple Wallet passes require PNG images at multiple resolutions to support different devices. All images must be PNG format.

### Supported Image Types

| Image Type | @1x (points) | @2x (points) | @3x (points) | Usage | Required |
|------------|------|---------|---------|-------|----------|
| `icon` | 29×29 | 58×58 | 87×87 | Displayed in Wallet list, lock screen | **Yes** |
| `logo` | Up to 160×50 | Up to 320×100 | Up to 480×150 | Logo display on pass | No |
| `strip` | 375×123 | 750×246 | 1125×369 | Full-width image on front (coupon, storeCard, eventTicket) | No |
| `strip` (eventTicket with thumbnail) | 375×84 | 750×168 | 1125×252 | Compact strip when thumbnail present | No |
| `thumbnail` | 90×90 | 180×180 | 270×270 | Small image (eventTicket, generic) | No |
| `background` | 180×220 | 360×440 | 540×660 | Background image (generic pass only, iOS <18) | No |
| `footer` | 286×15 | 572×30 | 858×45 | Bottom image on boardingPass | No |
| `personalizationLogo` | 150×40 | 300×80 | 450×120 | Logo for personalization flow | No |

### Event Ticket Poster Images (iOS 18+)

Additional images for poster-style event tickets:

| Image | Purpose | Specifications |
|-------|---------|-----------------|
| `background` | Full bleed background image | Full-screen dimensions. Automatically scaled. |
| Overlay logo | Logo overlaid on background | Should contrast with background. Max 480×480. |

### Icon Image Guidelines

- **Required for all passes**
- Minimum: 29×29 @1x
- Should have 1–2 pixel margin
- Avoid transparency; use solid background color
- Visible at small sizes; avoid fine detail
- Square format recommended

### Logo Image Guidelines

- Maximum dimensions: 160×50 @1x, 320×100 @2x, 480×150 @3x
- Transparent PNG recommended
- Keep aspect ratio consistent across resolutions
- Commonly company logo

### Strip Image Guidelines

- Full-width background image
- Dimensions: 375×123 @1x, 750×246 @2x, 1125×369 @3x
- Can be photo, gradient, or pattern
- Affected by `backgroundColor`, `suppressStripShine`
- eventTicket with thumbnail uses shorter dimensions (375×84)

### Thumbnail Guidelines

- Square format: 90×90 @1x, 180×180 @2x, 270×270 @3x
- Used on eventTicket and generic passes
- Often product photo or event poster

### Background Guidelines

- Only for generic pass type (iOS <18)
- Dimensions: 180×220 @1x, 360×440 @2x, 540×660 @3x
- Full-height, full-width background
- Obsolete with iOS 18+ poster support

### Footer Guidelines

- boardingPass only
- Horizontal image: 286×15 @1x, 572×30 @2x, 858×45 @3x
- Often airline logo or branding

### Image File Naming

Images are referenced in pass.json by **name without extension**:

```json
{
  "icon": "icon.png",           // Referenced as "icon"
  "logo": "logo.png",           // Referenced as "logo"
  "strip": "strip.png",         // Referenced as "strip"
  "thumbnail": "thumb.png",     // Referenced as "thumbnail"
  "background": "bg.png"        // Referenced as "background"
}
```

Place all image files in the **.pkpass bundle root** (alongside pass.json, manifest.json, signature).

For @2x and @3x variants, use naming convention:

```
icon.png           (@1x)
icon@2x.png        (@2x)
icon@3x.png        (@3x)

logo.png           (@1x)
logo@2x.png        (@2x)
logo@3x.png        (@3x)

strip.png          (@1x)
strip@2x.png       (@2x)
strip@3x.png       (@3x)
```

Reference in pass.json using base name only (without @2x/@3x suffix):

```json
{
  "icon": "icon.png",
  "logo": "logo.png",
  "strip": "strip.png"
}
```

System automatically selects correct resolution based on device.

### Image Format and Compression

- **Format**: PNG only
- **Color Space**: RGB or RGBA (sRGB preferred)
- **Alpha Channel**: Supported (transparency)
- **File Size**: Keep under 100 KB per image when possible
- **DPI**: Irrelevant (use pixel dimensions)
- **Interlacing**: Not required
- **Optimization**: Use PNG optimization tools (pngquant, pngcrush)

---

## 13. Bundle Structure

A `.pkpass` file is a ZIP archive containing all pass data, images, and cryptographic signature. The bundle contains:

```
mypass.pkpass (ZIP)
├── pass.json               (Pass data configuration)
├── manifest.json           (SHA-1 hashes of all files)
├── signature               (PKCS#7 detached signature)
├── icon.png               (Required image)
├── icon@2x.png
├── icon@3x.png
├── logo.png               (Optional images)
├── logo@2x.png
├── logo@3x.png
├── strip.png
├── strip@2x.png
├── strip@3x.png
├── thumbnail.png
├── thumbnail@2x.png
├── thumbnail@3x.png
├── en.lproj/              (Localization directories)
│   ├── pass.strings
│   ├── logo.png
│   └── strip.png
├── fr.lproj/
│   ├── pass.strings
│   ├── logo.png
│   └── strip.png
└── personalize.json       (Optional personalization config)
```

### Bundle Requirements

- **Format**: ZIP archive (no compression for some signing operations)
- **pass.json**: Required. Contains all pass configuration.
- **manifest.json**: Required. Contains SHA-1 hashes of all files except manifest.json and signature.
- **signature**: Required. Binary PKCS#7 detached signature of manifest.json.
- **Images**: At least icon required. All must be PNG format.
- **Locale directories**: .lproj format (e.g., en.lproj, fr.lproj). Optional but recommended for multi-language support.

---

## 14. Personalization

Allow users to personalize passes with custom information (e.g., name, membership number) after adding to Wallet.

### Configuration

Create a `personalize.json` file alongside `pass.json` in the `.pkpass` bundle:

```json
{
  "requiredPersonalizationFields": [
    "PKPassPersonalizationFieldName",
    "PKPassPersonalizationFieldPostalCode",
    "PKPassPersonalizationFieldEmailAddress",
    "PKPassPersonalizationFieldPhoneNumber"
  ],
  "description": "Personalize your membership card",
  "termsAndConditions": "By personalizing this pass, you agree to our terms of service."
}
```

### Personalization Field Types

| Field | Type | User Input | Description |
|-------|------|-----------|-------------|
| `PKPassPersonalizationFieldName` | String | Text input | User's full name |
| `PKPassPersonalizationFieldPostalCode` | String | Text input (numeric) | Postal/ZIP code |
| `PKPassPersonalizationFieldEmailAddress` | String | Text input (email) | Email address |
| `PKPassPersonalizationFieldPhoneNumber` | String | Text input (phone) | Phone number |

### Personalization Object

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `requiredPersonalizationFields` | Array | Yes | Array of field types user must provide. |
| `description` | String (Localizable) | No | Instruction text shown to user. Localizable. |
| `termsAndConditions` | String (Localizable) | No | Legal terms the user must accept. Localizable. |

### Personalization Flow

1. User adds pass with `personalize.json` to Wallet
2. Wallet displays personalization form with requested fields
3. User fills in their information
4. System collects data as `PersonalizationDictionary`
5. System sends POST request to your web service with personalization data
6. Server updates pass with personalized information
7. Updated pass sent back to device

### Personalization Data Structure

Device sends collected data to web service endpoint (custom, per your implementation):

```json
{
  "personalizedFieldValues": {
    "PKPassPersonalizationFieldName": "John Smith",
    "PKPassPersonalizationFieldPostalCode": "94107",
    "PKPassPersonalizationFieldEmailAddress": "john@example.com",
    "PKPassPersonalizationFieldPhoneNumber": "+1-555-123-4567"
  }
}
```

### Using Personalization in Pass

After receiving personalized data, update `pass.json` fields to include user information:

```json
{
  "pass.json": {
    "auxiliaryFields": [
      {
        "key": "name",
        "label": "Member Name",
        "value": "John Smith"
      },
      {
        "key": "postal",
        "label": "Postal Code",
        "value": "94107"
      }
    ]
  }
}
```

Then re-sign and return the updated pass to the device.

---

## 15. Localization

Support multiple languages and regions by creating language-specific directories within the `.pkpass` bundle.

### Directory Structure

```
mypass.pkpass (ZIP)
├── pass.json
├── manifest.json
├── signature
├── icon.png
├── en.lproj/
│   ├── pass.strings
│   ├── logo.png
│   └── strip.png
├── fr.lproj/
│   ├── pass.strings
│   ├── logo.png
│   └── strip.png
├── es.lproj/
│   └── pass.strings
└── de.lproj/
    └── pass.strings
```

### Language Codes

Use standard IETF language tags:

| Code | Language | Code | Language |
|------|----------|------|----------|
| `en` | English | `fr` | French |
| `es` | Spanish | `de` | German |
| `it` | Italian | `ja` | Japanese |
| `zh-Hans` | Simplified Chinese | `zh-Hant` | Traditional Chinese |
| `pt-BR` | Portuguese (Brazil) | `ru` | Russian |
| `ar` | Arabic | `ko` | Korean |

### Locale Directory Names

- **Language only**: `en.lproj`, `fr.lproj`, `es.lproj`
- **Language + Region**: `pt-BR.lproj`, `zh-Hans.lproj`, `zh-Hant.lproj`

### pass.strings File

Localizable strings file in each language directory. **UTF-16 encoding with BOM** required for non-ASCII characters.

**Format:**

```
/* Field Labels */
"key1" = "Localized Value 1";
"key2" = "Localized Value 2";

/* Longer descriptions */
"organizationName" = "Acme Corporation";
"description" = "Your store membership card";
```

**Example (en.lproj/pass.strings):**

```
"organizationName" = "Acme Store";
"description" = "Your loyalty card";
"gateLabel" = "Gate";
"seatLabel" = "Seat";
"departureLabel" = "Departs";
```

**Example (fr.lproj/pass.strings):**

```
"organizationName" = "Acme Magasin";
"description" = "Votre carte de fidélité";
"gateLabel" = "Porte";
"seatLabel" = "Siège";
"departureLabel" = "Départ";
```

### Localizable Keys

In `pass.json`, mark fields as localizable:

```json
{
  "organizationName": "Acme Store",
  "description": "Your loyalty card",
  "generic": {
    "primaryFields": [
      {
        "key": "balance",
        "label": "Balance",
        "value": "$100.00"
      }
    ]
  }
}
```

System automatically localizes `organizationName`, `description`, field `label` values, and all text strings based on device locale and available `.lproj` directories.

### Localized Images

Place region-specific images in language directories:

```
en.lproj/
├── pass.strings
├── logo.png        (English logo)
└── logo@2x.png

fr.lproj/
├── pass.strings
├── logo.png        (French logo)
└── logo@2x.png
```

Reference images by base name in `pass.json` (without language prefix):

```json
{
  "logo": "logo.png"
}
```

System selects locale-specific image if available; otherwise uses root image.

### Automatic Localization

Wallet automatically localizes:
- **Dates**: Format based on device locale (e.g., `12/31/2025` vs. `31/12/2025`)
- **Times**: 12-hour vs. 24-hour based on locale
- **Currency**: Symbol, decimal places based on locale
- **Numbers**: Thousands separators, decimal symbol

### Localization Example

**pass.json:**

```json
{
  "formatVersion": 1,
  "organizationName": "Acme Store",
  "description": "Membership Card",
  "storeCard": {
    "primaryFields": [
      {
        "key": "balance",
        "label": "Balance",
        "value": "150.50",
        "currencyCode": "USD"
      }
    ],
    "secondaryFields": [
      {
        "key": "expires",
        "label": "Expires",
        "value": "2025-12-31T23:59:59-08:00",
        "dateStyle": "PKDateStyleLong"
      }
    ]
  }
}
```

**en.lproj/pass.strings:**

```
"organizationName" = "Acme Store";
"description" = "Membership Card";
"Balance" = "Balance";
"Expires" = "Expires";
```

**fr.lproj/pass.strings:**

```
"organizationName" = "Magasin Acme";
"description" = "Carte d'adhésion";
"Balance" = "Solde";
"Expires" = "Expire";
```

**es.lproj/pass.strings:**

```
"organizationName" = "Tienda Acme";
"description" = "Tarjeta de membresía";
"Balance" = "Saldo";
"Expires" = "Vence";
```

Device with French locale receives French strings; Spanish device receives Spanish strings, etc.

---

## 16. Distribution

Deploy and deliver `.pkpass` files to users through various channels.

### Distribution Methods

1. **Email** – Send `.pkpass` as email attachment with MIME type `application/vnd.apple.pkpass`.

2. **Web Download** – Host `.pkpass` on web server with proper Content-Type headers. Users download and open in Wallet.

3. **QR Code** – Generate QR code linking to `.pkpass` download URL. User scans with iPhone camera to download.

4. **iOS App** – Use native PassKit framework (PKAddPassesViewController) to add passes directly from your iOS app.

5. **AirDrop** – Share `.pkpass` via AirDrop on macOS/iOS between devices.

6. **Multi-Pass Bundle** – Distribute multiple passes in a single `.pkpasses` file (ZIP of multiple `.pkpass` files). MIME type: `application/vnd.apple.pkpasses`.

7. **Universal Links** – Use Apple Universal Links to handle pass additions seamlessly in iOS apps. Implement `.well-known/apple-app-site-association` for deep linking.

### Security Considerations for Distribution

- **HTTPS only** – Always use HTTPS to prevent interception
- **Authenticate users** – Require authentication for sensitive passes
- **Rate limiting** – Prevent abuse of pass download endpoints
- **Expiration** – Set reasonable expiration dates on download URLs
- **Logging** – Log all pass downloads for security audits
- **Signature validation** – Always sign passes cryptographically

### Accessibility for Distribution

- **Alt text** for QR codes: "Scan to download boarding pass"
- **Direct download link** as alternative to QR code
- **Email accessibility** – Ensure plain-text link in email
- **Descriptive filenames** – Use meaningful pass names

---

## Additional Resources

- **Apple Developer**: https://developer.apple.com/wallet/
- **Pass Kit Framework**: https://developer.apple.com/documentation/passkit
- **Apple Wallet Design Guidelines**: https://developer.apple.com/design/human-interface-guidelines/wallet
- **ISO 8601 DateTime**: https://en.wikipedia.org/wiki/ISO_8601

---

**Document Version:** 2.0 (Revised)
**Last Updated:** April 8, 2026
**For:** Claude Code Apple Wallet Pass Generation Skill
