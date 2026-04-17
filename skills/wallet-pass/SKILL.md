---
name: wallet-pass
description: |
  Comprehensive knowledge base for Apple Wallet and Google Wallet digital passes. Covers ALL pass types, field definitions, visual layout rules, image specifications, barcode formats, semantic tags, localization, and cross-platform differences. Use this skill whenever the user mentions Apple Wallet, Google Wallet, .pkpass, PassKit, digital pass, mobile pass, wallet pass, loyalty card, event ticket, boarding pass, coupon, gift card, store card, transit pass, or is building any kind of digital pass for iPhone, Apple Watch, or Android. Also trigger when the user asks about pass.json structure, pass field layout, pass types, what fields a pass can have, image sizes for passes, barcode formats, or how passes look on each platform. Use even if only one platform is mentioned — it covers both and highlights cross-platform differences.
---

# Wallet Pass Knowledge Base (Apple + Google)

Complete reference for digital wallet pass types, structures, fields, images, barcodes, and visual layout rules on both Apple Wallet and Google Wallet. This skill is about **what passes can contain and how to configure them** — the data structures, field definitions, enums, and design rules.

## When to Read Reference Files

This SKILL.md provides the high-level overview and cross-platform mapping. For complete field-by-field definitions, read the reference files:

- **`references/apple-wallet.md`** — Every Apple pass.json field, all 5 pass types with field limits, PassFieldContent properties, semantic tags (100+ fields), image specs, relevance, localization, personalization, .pkpass bundle structure
- **`references/google-wallet.md`** — Every Google class/object field for all 7 types, all sub-objects and enums, shared fields, image specs, barcode details, rotating barcodes, pass constraints
- **`references/cross-platform.md`** — Side-by-side field mapping, barcode comparison, image comparison, feature matrix, color format conversion, key differences and similarities

Always read the relevant reference before generating pass configurations. If building for both platforms, read all three.

---

## Platform Architecture Overview

### Apple Wallet
A `.pkpass` is a signed ZIP bundle containing a `pass.json` file (all pass data and configuration), images, and optional localizations. Each pass is self-contained — there's no server-side template concept. Five pass types, each with a distinct visual layout.

### Google Wallet
Google uses a **Class/Object** model. A **Class** is a shared template defining the pass design and shared data (e.g., event name, venue). An **Object** is an individual pass instance with user-specific data (e.g., seat, barcode). Seven pass types, each with type-specific fields on both class and object.

### Key Structural Differences

| Aspect | Apple | Google |
|--------|-------|--------|
| Data format | Single `pass.json` in a ZIP | Separate Class (template) + Object (instance) JSON |
| Pass types | 5 | 7 |
| Images | Bundled as PNG files in ZIP | Referenced via HTTPS URLs |
| Barcodes | 4 formats | 13 formats + rotating (TOTP) |
| Colors | `rgb(r, g, b)` strings | `#rrggbb` hex strings |
| Localization | `.lproj` directories with `pass.strings` | `LocalizedString` objects with `translatedValues` |
| Semantic metadata | 100+ semantic tag fields | No equivalent |
| Relevance | Locations (10), beacons, dates | merchantLocations (10), validTimeInterval |
| Field layout | headerFields, primaryFields, secondaryFields, auxiliaryFields, backFields | textModulesData, imageModulesData, linksModuleData + type-specific structured fields |

---

## Pass Type Mapping

| Use Case | Apple Type | Google Type | Notes |
|----------|-----------|-------------|-------|
| Airline boarding | `boardingPass` (transitType: Air) | `FlightClass/Object` | Apple uses one type for all transit; Google has dedicated Flight |
| Train/bus/ferry | `boardingPass` (transitType: Train/Bus/Boat) | `TransitClass/Object` | Google separates transit from flights |
| Concert/sports/conference | `eventTicket` | `EventTicketClass/Object` | Apple supports poster-style display |
| Loyalty/membership | `storeCard` | `LoyaltyClass/Object` | Apple uses storeCard for both loyalty and gift cards |
| Gift card | `storeCard` | `GiftCardClass/Object` | Google has a dedicated type with balance/pin fields |
| Coupon/discount/offer | `coupon` | `OfferClass/Object` | Similar purpose, different field structures |
| Generic/other | `generic` | `GenericClass/Object` | Google has `genericType` enum for sub-categorization |

---

## Apple Wallet — Structure Summary

### pass.json Required Top-Level Fields
Every Apple pass must include: `formatVersion` (always 1), `passTypeIdentifier`, `serialNumber`, `teamIdentifier`, `organizationName`, `description`.

### Visual Appearance
Colors as `rgb(r, g, b)`: `backgroundColor`, `foregroundColor`, `labelColor`. Also: `logoText`, `suppressStripShine`. Poster event tickets add: `eventLogoText`, `footerBackgroundColor`, `suppressHeaderDarkening`, `useAutomaticColors`.

### Five Pass Types — Field Groups
Each type contains field arrays: `headerFields`, `primaryFields`, `secondaryFields`, `auxiliaryFields`, `backFields`. Poster event tickets also get `additionalInfoFields`.

**Field limits per type:**

| Type | Header | Primary | Secondary | Auxiliary | Back |
|------|--------|---------|-----------|-----------|------|
| boardingPass | 2 | 2 | 5 | 5 | unlimited |
| coupon | 1 | 1 | 4 | 4 | unlimited |
| eventTicket | 3 | 2 (1 poster) | 4 | 4 | unlimited |
| storeCard | 3 | 1 | 4 | 4 | unlimited |
| generic | 3 | 1 | 4 | 4 | unlimited |

### PassFieldContent (Each Individual Field)
Required: `key` (unique string), `value` (string, number, or ISO 8601 date).
Optional: `label`, `attributedValue` (HTML links only), `changeMessage` (with `%@`), `textAlignment`, `currencyCode`, `numberStyle`, `dateStyle`, `timeStyle`, `ignoresTimeZone`, `isRelative`, `dataDetectorTypes` (back fields only), `row`, `semantics`.

### Boarding Pass Specifics
Requires `transitType`: `PKTransitTypeAir`, `PKTransitTypeTrain`, `PKTransitTypeBus`, `PKTransitTypeBoat`, `PKTransitTypeGeneric`. Supports 13+ action URLs (changeSeatURL, entertainmentURL, trackBagsURL, etc.).

### Event Ticket Specifics
Supports poster-style display. Has 14+ action URLs (accessibilityURL, merchandiseURL, transferURL, etc.).

### Barcodes
`barcodes` array with objects: `format`, `message`, `messageEncoding`, `altText`.
Formats: `PKBarcodeFormatQR`, `PKBarcodeFormatPDF417`, `PKBarcodeFormatAztec`, `PKBarcodeFormatCode128`.

### Semantic Tags
100+ machine-readable fields under `semantics` key for system features (Do Not Disturb, Maps, Siri). Categories: general, event/sports, transit/boarding, balance.

### Images
Bundled as PNGs with @1x/@2x/@3x variants. Required: `icon.png`. Others: `logo`, `strip`, `thumbnail`, `background`, `footer`, `personalizationLogo`.

### Other Features
Relevance (locations, dates, beacons), expiration/voiding, grouping, NFC, personalization, localization via `.lproj` directories, associated apps.

**For complete field definitions, see `references/apple-wallet.md`.**

---

## Google Wallet — Structure Summary

### Class/Object Model
Every Google pass has a Class (shared template) and Object (individual instance). Classes need `reviewStatus: "UNDER_REVIEW"` before objects can be saved. Both class and object can have textModulesData, imageModulesData, linksModuleData (up to 10 each, all displayed).

### Seven Pass Types

**Generic** — Most flexible. Object has `cardTitle`, `subheader`, `header` (all LocalizedString), `logo`, `wideLogo`, `genericType` enum (14 values including GENERIC_SEASON_PASS, GENERIC_GYM_MEMBERSHIP, GENERIC_PARKING_PASS, etc.).

**Loyalty** — Class: `programName`, `programLogo`, `accountNameLabel`, `accountIdLabel`, `rewardsTier`, `secondaryRewardsTier`, `discoverableProgram`. Object: `accountName`, `accountId`, `loyaltyPoints` (with label + balance), `secondaryLoyaltyPoints`.

**Offer** — Class: `title`, `provider`, `titleImage`, `details`, `finePrint`, `redemptionChannel` enum (INSTORE, ONLINE, BOTH, TEMPORARY_PRICE_REDUCTION). Object: references class, adds user-specific barcode/state.

**Gift Card** — Class: `merchantName`, `programLogo`, `pinLabel`, `cardNumberLabel`. Object: `cardNumber` (required), `pin`, `balance` (Money), `balanceUpdateTime`, `eventNumber`.

**Event Ticket** — Class: `eventName` (required), `venue` (name + address), `dateTime` (doorsOpen, start, end), `logo`, label fields for seat/row/section/gate/confirmation. Object: `seatInfo` (EventSeat), `reservationInfo`, `ticketHolderName`, `ticketNumber`, `ticketType`, `faceValue`.

**Flight** — Class: `flightHeader` (required, with carrier info + flight number), `origin`/`destination` (AirportInfo with IATA code, terminal, gate), date/time fields (ISO 8601 WITHOUT timezone offset — local airport time), `flightStatus`, `boardingAndSeatingPolicy`. Object: `passengerName` (required), `boardingAndSeatingInfo` (group, seat, class, door), `reservationInfo` (required, with confirmation code + frequent flyer).

**Transit** — Class: `transitOperatorName`, `logo`, `transitType` (required: BUS, RAIL, TRAM, FERRY, OTHER), `activationOptions`, 19 custom label fields. Object: `ticketNumber`, `passengerType`, `ticketStatus`/`customTicketStatus`, `concessionCategory`/`customConcessionCategory`, `ticketRestrictions`, `purchaseDetails`, `ticketLeg`/`ticketLegs` (with TicketSeat, fareClass, departure/arrival), `tripType` (required: ROUND_TRIP, ONE_WAY), `activationStatus`.

### Shared Fields (All Types)
**On classes:** `id`, `issuerName`, `reviewStatus`, `messages` (max 10), `hexBackgroundColor`, `heroImage`, `imageModulesData`, `textModulesData`, `linksModuleData`, `enableSmartTap`, `callbackOptions`, `securityAnimation`, `viewUnlockRequirement`, `classTemplateInfo`, `valueAddedModuleData` (max 10), `merchantLocations` (max 10).

**On objects:** `id`, `classId`, `state` (ACTIVE, COMPLETED, EXPIRED, INACTIVE), `barcode`, `rotatingBarcode`, `messages` (max 10), `validTimeInterval`, `smartTapRedemptionValue`, `imageModulesData`, `textModulesData`, `linksModuleData`, `heroImage`, `groupingInfo`, `passConstraints`, `saveRestrictions`, `linkedObjectIds`, `valueAddedModuleData` (max 10), `merchantLocations` (max 10).

### Barcodes
13 formats: `QR_CODE`, `AZTEC`, `PDF_417`, `CODE_128`, `CODE_39`, `CODE_93`, `CODABAR`, `DATA_MATRIX`, `EAN_8`, `EAN_13`, `ITF_14`, `UPC_A`, `TEXT_ONLY`. Plus TOTP-based `RotatingBarcode`.

### Images
All hosted at HTTPS URLs (not bundled). `logo`: 800x800px, `wideLogo`: 1600x800px, `heroImage`: 1032x336px, `imageModulesData`: 1860x930px.

### Key Data Types
`LocalizedString` (defaultValue + translatedValues), `Money` (micros + currencyCode), `Image` (sourceUri + contentDescription), `TimeInterval` (start + end DateTime).

**For complete field definitions, see `references/google-wallet.md`.**

---

## Barcode Format Comparison

| Format | Apple Constant | Google Constant |
|--------|---------------|-----------------|
| QR Code | `PKBarcodeFormatQR` | `QR_CODE` |
| PDF417 | `PKBarcodeFormatPDF417` | `PDF_417` |
| Aztec | `PKBarcodeFormatAztec` | `AZTEC` |
| Code 128 | `PKBarcodeFormatCode128` | `CODE_128` |
| Code 39 | not supported | `CODE_39` |
| Code 93 | not supported | `CODE_93` |
| Codabar | not supported | `CODABAR` |
| Data Matrix | not supported | `DATA_MATRIX` |
| EAN-8 | not supported | `EAN_8` |
| EAN-13 | not supported | `EAN_13` |
| ITF-14 | not supported | `ITF_14` |
| UPC-A | not supported | `UPC_A` |
| Text Only | not supported | `TEXT_ONLY` |
| Rotating (TOTP) | not supported | `RotatingBarcode` object |

---

## Image Comparison

| Purpose | Apple | Google |
|---------|-------|--------|
| App/notification icon | `icon.png` 29x29pt (required) | No equivalent |
| Logo | `logo.png` up to 160x50pt | `logo` 800x800px (square) |
| Wide logo | not supported | `wideLogo` 1600x800px |
| Banner/hero | `strip.png` 375x123pt | `heroImage` 1032x336px |
| Thumbnail | `thumbnail.png` 90x90pt | No equivalent |
| Background | `background.png` 180x220pt | `hexBackgroundColor` (color only) |
| Footer | `footer.png` 286x15pt | No equivalent |
| Custom images | not supported | `imageModulesData` 1860x930px |

Apple bundles PNGs at @1x/@2x/@3x in the ZIP. Google references external HTTPS URLs.

---

## Color Format

Apple uses CSS-style RGB: `rgb(255, 255, 255)`
Google uses hex: `#ffffff` or shorthand `#fff`

---

## Key Rules and Constraints

### Apple
- `formatVersion` must be `1`
- `passTypeIdentifier` must match the signing certificate
- `icon.png` is required for all pass types
- Use `barcodes` array (not deprecated singular `barcode` key)
- `changeMessage` must contain `%@` placeholder for the new value
- `serialNumber` must be unique per `passTypeIdentifier`
- Same `groupingIdentifier` + `passTypeIdentifier` + type = grouped in Wallet
- Dates are ISO 8601 with timezone
- Colors are `rgb(r, g, b)` format

### Google
- All IDs: `issuerId.identifier` format (alphanumeric, `.`, `_`, `-` only)
- Max 10 messages, 10 valueAddedModuleData, 10 merchantLocations per resource
- `notifyPreference` is ephemeral — set on every update request
- `locations` field deprecated — use `merchantLocations`
- `kind`, `version` fields deprecated everywhere
- Flight dates: ISO 8601 WITHOUT timezone offset (local airport time)
- Transit `ticketLeg`/`ticketLegs` mutually exclusive
- Transit `ticketStatus`/`customTicketStatus` mutually exclusive
- Transit `concessionCategory`/`customConcessionCategory` mutually exclusive
- Object `state`: ACTIVE (visible), COMPLETED (used), EXPIRED (archived), INACTIVE (hidden)
- Classes need `reviewStatus: "UNDER_REVIEW"` before objects can be saved

**For full cross-platform comparison, see `references/cross-platform.md`.**
