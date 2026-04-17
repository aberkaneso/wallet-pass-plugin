# Cross-Platform Apple Wallet and Google Wallet Pass Reference

A comprehensive mapping and comparison guide for developers working with both Apple Wallet (.pkpass) and Google Wallet passes.

## 1. Pass Type Mapping

| Apple Pass Type | Google Equivalent | Key Differences |
|---|---|---|
| **boardingPass** (transitType: Air) | FlightClass / FlightObject | Apple: standalone fields (origin, destination, gate, seat); Google: class defines structure, object holds values. Apple: string values with semantic tags; Google: structured objects (FlightHeader, BoardingAndSeatingInfo). Apple can have multiple barcodes; Google: 1 primary barcode or 1 rotating. |
| **boardingPass** (transitType: Train) | TransitClass / TransitObject (classType: TRANSIT) | Apple: uses auxiliaryFields for station codes and track; Google: uses transitDetails.lines and boardingAndSeatingInfo. Apple: has predefined field labels; Google: custom label configuration. |
| **boardingPass** (transitType: Bus) | TransitClass / TransitObject (classType: TRANSIT) | Similar to train. Google allows rotating barcodes for dynamic passes; Apple does not. |
| **boardingPass** (transitType: Boat) | TransitClass / TransitObject (classType: TRANSIT) | Apple: uses secondaryFields for departure info; Google: uses transitDetails.stopoverStations. |
| **eventTicket** | EventTicketClass / EventTicketObject | Apple: organizer/eventType in primaryFields; Google: structured eventName, venue, dateTime, seatInfo objects. Apple: supports up to 100 semantic fields; Google: no semantic tags. |
| **storeCard** (loyalty) | LoyaltyClass / LoyaltyObject | Apple: fields map directly; Google: class defines program structure, object holds account data, loyaltyPoints, barcode. Google's class-based approach enables bulk updates. |
| **storeCard** (gift card) | GiftCardClass / GiftCardObject | Apple: balance in auxiliaryFields; Google: explicit cardNumber, balance, pin, currency fields. Google has dedicated gift card class. |
| **coupon** | OfferClass / OfferObject | Apple: coupon-specific fields; Google: offer details including title, provider, restrictions. Both support barcode and time validity. |
| **generic** | GenericClass / GenericObject | Apple: most flexible, minimal constraints; Google: simpler pass type with header, subheader, details. Apple supports more semantic metadata. |

## 2. Field Mapping Tables

### Event Ticket

| Apple Field | Google Equivalent | Notes |
|---|---|---|
| `primaryFields[0].value` | `eventName` | Event name/title |
| `secondaryFields` (venue info) | `venue.name`, `venue.address` | Apple: multiple fields; Google: structured venue object |
| `auxiliaryFields` (seat/row/section) | `seatInfo.seat`, `seatInfo.row`, `seatInfo.section`, `seatInfo.gate` | Apple: individual field values; Google: structured seatInfo object |
| `barcodes[0]` | `barcode` | Same barcode data, different format structure |
| `semantics.eventStartDate` | `dateTime.start` | ISO 8601 timestamp |
| `semantics.eventEndDate` | `dateTime.end` | ISO 8601 timestamp |
| `organizationName` | `issuerName` | Event organizer |
| `labelColor` + `labelForegroundColor` | Defined in class CSS | Apple: RGB values; Google: hex colors |
| `foregroundColor` | `widgetColor` (custom) | Pass text color |
| `backgroundColor` | `hexBackgroundColor` | Pass background color |
| `stripImage` | `heroImage` | Apple: 375x123pt; Google: 1032x336px HTTPS URL |
| `logo` | `logo` | Apple: bundled PNG; Google: HTTPS URL |

### Boarding Pass / Flight

| Apple Field | Google Equivalent | Notes |
|---|---|---|
| `boardingPass.transitType` | Determines FlightClass vs TransitClass | Apple: enumeration; Google: class selection |
| `primaryFields[0]` (origin code) | `flightHeader.origin.airportIata` | Origin airport code |
| `primaryFields[1]` (destination code) | `flightHeader.destination.airportIata` | Destination airport code |
| `auxiliaryFields` (gate, seat, boarding time) | `boardingAndSeatingInfo.gate`, `boardingAndSeatingInfo.seat`, `boardingAndSeatingInfo.boardingTime` | Apple: flexible field positioning; Google: structured objects |
| `boardingPass.boardingSequence` | `boardingAndSeatingInfo.boardingSequence` | Sequential boarding number |
| `boardingPass.operatingCarrierPNR` | `flightHeader.operatingCarrierCode` | Airline code |
| `semantics.departureStationName` | `flightHeader.origin.terminal` | Departure details |
| `semantics.arrivalStationName` | `flightHeader.destination.terminal` | Arrival details |
| `secondaryFields` (flight number) | `flightHeader.flightNumber` | Structured flight info |
| `barcodes[0]` | `barcode` | Barcode object (1 primary) |
| `serialNumber` | `id` | Unique pass identifier |

### Loyalty / Store Card

| Apple Field | Google Equivalent | Notes |
|---|---|---|
| `storeCard.primaryFields` | `programName` | Loyalty program name (in class) |
| `barcode` | `barcode` | Program barcode |
| `backFields[0]` (account number) | `accountId` (LoyaltyObject) | Customer ID |
| `backFields[1]` (points balance) | `loyaltyPoints` (LoyaltyObject) | Points value and label |
| `secondaryFields` | `accountName` (LoyaltyObject) | Cardholder name |
| `logo` | `logo` (in class) | Program logo |
| `labelColor` | Not directly mapped | Google uses class-level styling |
| `description` | `description` | Program description |
| `expirationDate` | `validTimeInterval.end` | Card expiration |

### Coupon / Offer

| Apple Field | Google Equivalent | Notes |
|---|---|---|
| `coupon.auxiliaryFields[0].value` | `title` | Offer description |
| `coupon.primaryFields[0]` (discount) | `details` | Discount amount/percentage |
| `coupon.secondaryFields` (restrictions) | `restrictionDates`, `restrictions` | Validity and limitations |
| `organizationName` | `provider` | Issuing retailer |
| `barcode` | `barcode` | Coupon code barcode |
| `expirationDate` | `validTimeInterval.end` | Offer expiration |
| `backgroundColor` | `hexBackgroundColor` | Offer color |

### Gift Card

| Apple Field | Google Equivalent | Notes |
|---|---|---|
| `storeCard.primaryFields[0]` | `cardNumber` | Card number (may be masked) |
| `auxiliaryFields` (balance) | `balance` | Remaining balance with currency |
| `auxiliaryFields` (PIN) | `pin` | Card PIN (if applicable) |
| `barcode` | `barcode` | Redemption barcode |
| `organizationName` | `issuerName` | Retailer name |
| `expirationDate` | `validTimeInterval.end` | Card expiration |
| `logo` | `logo` | Retailer logo |
| Apple: no PIN field | `pin` (explicit) | Google has dedicated pin field |

### Generic

| Apple Field | Google Equivalent | Notes |
|---|---|---|
| `generic.primaryFields` | `header` | Main content/title |
| `generic.secondaryFields` | `subheader` | Secondary information |
| `generic.auxiliaryFields` | `textModulesData` | Additional details |
| `generic.backFields` | `linksModuleData` | Footer and links |
| `barcode` | `barcode` | Pass barcode (optional) |
| `logo` | `logo` | Pass logo |
| `foregroundColor` | `widgetColor` | Text color |
| `backgroundColor` | `hexBackgroundColor` | Background color |

## 3. Barcode Format Mapping

| Barcode Type | Apple Constant | Google Constant | Notes |
|---|---|---|---|
| QR Code | `PKBarcodeFormatQR` | `QR_CODE` | Most common, both fully support |
| PDF417 | `PKBarcodeFormatPDF417` | `PDF_417` | 2D format, common for boarding |
| Aztec | `PKBarcodeFormatAztec` | `AZTEC` | 2D format |
| Code 128 | `PKBarcodeFormatCode128` | `CODE_128` | 1D format |
| Code 39 | N/A | `CODE_39` | Google only |
| Code 93 | N/A | `CODE_93` | Google only |
| CODABAR | N/A | `CODABAR` | Google only |
| Data Matrix | N/A | `DATA_MATRIX` | Google only |
| EAN-8 | N/A | `EAN_8` | Google only |
| EAN-13 | N/A | `EAN_13` | Google only |
| ITF-14 | N/A | `ITF_14` | Google only |
| UPC-A | N/A | `UPC_A` | Google only |
| Text-Only | N/A | `TEXT_ONLY` | Google only, no barcode image |

**Rotating Barcodes:**
- **Apple:** Does not support rotating/dynamic barcodes
- **Google:** Supports rotating barcodes with TOTP (time-based one-time password). Barcode updates every 30 seconds. Requires `BarcodeValue.alternateText` field.

## 4. Image Comparison

| Purpose | Apple | Google | Notes |
|---|---|---|---|
| **Logo** | PNG file bundled in .pkpass; @1x, @2x, @3x variants; max 160x50 points; ≤100 KB per variant | External HTTPS URL; 800x800 pixels; ≤1 MB | Apple: point-based scaling; Google: pixel-based. Apple: bundled; Google: hosted. |
| **Hero/Strip** | strip.png bundled; 375x123 points; ≤150 KB | heroImage HTTPS URL; 1032x336 pixels; ≤1 MB | Large banner across top of pass. Apple @2x/@3x needed. |
| **Thumbnail** | thumbnail.png bundled; 90x90 points; ≤150 KB | No equivalent | Apple: for Wallet list view. Google has no thumbnail. |
| **Background** | background.png bundled; 180x220 points (card); ≤150 KB | hexBackgroundColor (#rrggbb) or no image | Apple: detailed image; Google: solid color or no background. |
| **Footer** | footer.png bundled; 286x15 points; ≤150 KB | No equivalent (use linksModuleData) | Apple: bottom strip. Google: use text links instead. |
| **Icon** | icon.png; 29x29 points; required; Apple-specific | No equivalent | Apple: small corner icon. Google has no icon requirement. |
| **Tint/Accent** | No image; labelColor RGB | widgetColor (custom) or class-level CSS | Apple: field label color; Google: widget accent color. |

**Bundling:**
- **Apple:** All images bundled in ZIP file (.pkpass), referenced by filename
- **Google:** All images hosted externally, referenced by HTTPS URL

## 5. Feature Comparison Matrix

| Feature | Apple Wallet | Google Wallet | Notes |
|---|---|---|---|
| **Pass Format** | Signed ZIP archive (.pkpass) | REST API + JWT + hosted images | Apple: file-based; Google: API-based |
| **Signing Method** | PKCS#7 certificate chain | JWT (RS256 algorithm) | Apple: X.509; Google: RSA private key |
| **Certificate/Key** | Apple Developer certificate + WWDR intermediate | Google service account private key | Different PKI infrastructure |
| **Offline Support** | Full (pass works completely offline) | Partial (requires network for initial save, then works offline) | Apple wins for offline-first scenarios |
| **Template Model** | None; each pass is independent | Class + Object pattern | Google enables bulk updates via class |
| **Max Barcodes** | Multiple in array; first is primary | 1 primary barcode or 1 rotating barcode | Apple: more flexibility; Google: rotation capability |
| **Barcode Formats** | 4 formats (QR, PDF417, Aztec, Code128) | 13+ formats; rotating (TOTP) | Google more comprehensive |
| **Rotating Barcodes** | Not supported | Supported (TOTP, 30-second refresh) | Google: dynamic codes; Apple: static |
| **NFC Support** | Yes (nfc field in pass) | Smart Tap (partnership required) | Apple: native; Google: limited/partnership |
| **Location Relevance** | Up to 10 locations + iBeacon support | Up to 10 merchantLocations | Both similar, Apple has beacon option |
| **Date Relevance** | relevantDates array | validTimeInterval (start + end) | Apple: multiple dates; Google: range |
| **Beacon Support** | Yes (iBeacon, Eddystone) | No | Apple exclusive |
| **Pass Grouping** | groupingIdentifier | groupingInfo.groupingId | Same concept, different naming |
| **Update Mechanism** | Push notification + web service (5 endpoints) | REST API PATCH request | Apple: pull/push hybrid; Google: push only |
| **Update Notifications** | changeMessage per field; silent updates possible | notifyPreference (ephemeral flag) | Apple: detailed; Google: simple flag |
| **Pass Expiration** | expirationDate field | state: EXPIRED or validTimeInterval | Both supported |
| **Pass Voiding** | voided: true flag | state: COMPLETED | Apple: simple flag; Google: state field |
| **Pass States** | ACTIVE / voided / expired | ACTIVE / COMPLETED / EXPIRED / INACTIVE | Google: 4 states; Apple: 3 states |
| **Personalization** | personalizePass flow; user's name embedded | No equivalent | Apple exclusive; adds personal touch |
| **Localization** | .lproj directories + pass.strings file | LocalizedString objects in JSON | Both support multi-language |
| **Color Format** | rgb(r, g, b) format | #rrggbb or rgb (hex preferred) | Different formats; Apple: `rgb(r, g, b)` vs Google: `#rrggbb` |
| **Associated Apps** | associatedStoreIdentifiers array | appLinkData object | Link pass to app; both support |
| **Max Locations** | 10 locations + 10 iBeacons | 10 merchantLocations | Apple: higher with beacons |
| **Semantic Tags** | 100+ semantic fields (eventType, airline, etc.) | No semantic tags | Apple: rich metadata; Google: none |
| **Pass Bundling** | Multiple .pkpass in .pkpasses ZIP | Multiple objects in single JWT | Both support multiple passes |
| **Back of Pass** | backFields array | textModulesData + linksModuleData | Apple: field-based; Google: modular |
| **Field Labels** | Fixed per PassFieldContent | Custom labels (Transit has 19 label options) | Google: more customizable |
| **Web Service Updates** | Requires hosted web service with 5 endpoints | Simple REST API endpoints | Apple: more complex; Google: simpler |
| **Development Complexity** | Higher (cryptographic signing, web service) | Medium (JWT generation, REST API) | Apple requires more infrastructure |
| **Testing** | Requires iOS device or simulator | Can use save URL in browser | Apple: device-required; Google: browser-testable |

## 6. Color Format Note

Apple uses `rgb(r, g, b)` format (example: `rgb(255, 0, 128)`), while Google uses hexadecimal `#rrggbb` format (example: `#ff0080`). When porting passes between platforms, color values must be converted between these two formats.

## 8. Key Differences Summary

### Architecture & Implementation

**Apple Wallet** uses a **file-based approach** while **Google Wallet** uses an **API-based architecture**. Apple passes are self-contained .pkpass files (signed ZIP archives) that can be distributed directly, installed offline, and work independently. Google passes are instances of remotely-hosted Classes, meaning the Class (template) lives on Google's servers and Objects (instances) reference it, enabling bulk updates—change the Class and all Objects update simultaneously.

### Signing & Certificates

Apple requires you to sign passes locally using PKCS#7 cryptographic signing with an Apple Developer certificate. You must implement or use a library for signature generation and pass encryption. Google uses JWT (RS256) signatures with a service account private key from Google Cloud Console. Google's approach is simpler and requires no certificate authority involvement.

### Update Mechanisms

Apple employs a **pull-push hybrid** model: you host a web service with 5 specific endpoints (registrations, devices, updates, etc.) and send push notifications when passes change. Users' devices periodically check your web service. Google uses **simple REST API PATCH** requests—just update the Object or Class via their REST API, and Google sends the push notification. Apple's approach is more complex but offers more control; Google's is simpler but less flexible.

### Metadata & Semantics

Apple supports **100+ semantic fields** (e.g., `eventType`, `airlines`, `departureGate`, `transitStopName`) that provide rich structured meaning for pass content. This enables better rendering and interactions. Google has **no semantic tag system**—passes are just objects with custom key-value pairs. If you need semantics in Google, you must implement your own validation and interpretation.

### Barcode Formats

Both support QR, PDF417, Aztec, and Code 128. Google additionally supports 9 other formats (Code 39, EAN-13, UPC-A, etc.) and **rotating barcodes** using TOTP (time-based one-time passwords that refresh every 30 seconds). Apple supports only static barcodes. For high-security boarding passes or concert tickets, Google's rotating barcodes provide an advantage.

### Images & Resources

Apple **bundles all images** in the .pkpass ZIP file using point-based sizing with @1x/@2x/@3x variants for different screen densities. Google **hosts images externally** via HTTPS URLs with pixel-based dimensions. Apple's bundling means larger file sizes but guaranteed availability; Google's approach keeps passes small but requires external image infrastructure.

### Offline Capability

Apple passes **work fully offline** once installed—they're complete, signed files with all data embedded. Google passes **require network access to initially save** to the wallet, but work offline afterward. For connectivity-challenged scenarios (airports, remote areas), Apple has an advantage.

### Personalization

Apple offers a **personalizePass flow** where you can request the user's name and other personal info before adding the pass, then embed it in the pass. Google has **no equivalent**—you cannot request or embed personal data. For loyalty cards or event tickets where personalization adds value, Apple is the only option.

### Localization

Both support multi-language passes. Apple uses .lproj directories and pass.strings files (one per language). Google uses LocalizedString objects directly in JSON. Both approaches work; Google is simpler (no file structure needed).

### Pass States

Apple has 3 states: active, voided, expired. Google has 4: ACTIVE, COMPLETED (voided), EXPIRED, INACTIVE (disabled but not expired). Google's INACTIVE state is useful for temporarily disabling passes without expiring them.

### NFC Support

Apple has **native NFC support** via the `nfc` field in passes. Google's support is limited to **Smart Tap**, which requires a partnership with Google and specific backend infrastructure. For general NFC use, Apple is far simpler.

### Beacon Support

Apple supports **iBeacon and Eddystone** beacons for location-triggered pass relevance. Google has **no beacon support**. For retail applications using Bluetooth beacons, Apple is required.

## 9. Similarities Summary

Despite architectural differences, Apple and Google Wallet passes share significant common ground:

### Barcodes

Both support **QR, PDF417, Aztec, and Code 128** barcodes, the four most common formats across retail, events, and travel.

### Location Relevance

Both allow up to **10 locations** with latitude/longitude and optional relevance text. When a user is near a location, the pass can automatically become relevant in their wallet.

### Time-Based Relevance

Both support **time-based validity**: start dates, end dates, and expiration. Passes can become active/inactive based on current time.

### Pass Grouping

Both allow **grouping related passes together** in the wallet using a grouping identifier. Loyalty cards for the same program or event tickets from the same organizer can be grouped.

### Push Notifications

Both support **push notifications when passes update**. When you change a pass, the system notifies the user's device, and the updated pass appears in their wallet.

### Localization

Both support **multi-language passes** with different content for different locales. Useful for international passes.

### App Linking

Both support **linking passes to companion mobile apps** (Apple: associatedStoreIdentifiers; Google: appLinkData). Users can tap to open your app from the pass.

### JSON Data Format

Both use **JSON as the core data format**. While structures differ, both are JSON-based, making parsing and generation relatively straightforward.

### Developer Accounts & Credentials

Both require **developer accounts and cryptographic credentials** (Apple: Developer Account + certificates; Google: Google Cloud Console + service account). Both have authentication infrastructure.

### Class/Template Concept

Although implemented differently, both enable **template-like behavior**: Apple via common pass structure conventions; Google via explicit Class/Object separation. This allows consistency across multiple passes of the same type.

---

## Quick Reference: When to Use Which Platform

| Scenario | Recommendation | Why |
|---|---|---|
| Offline-first passes | Apple | Full offline support; Google requires network to save |
| Rotating/dynamic barcodes | Google | Apple doesn't support rotating barcodes |
| Personalized passes (name embedding) | Apple | No Google equivalent for personalization |
| Beacon-triggered passes | Apple | Google has no beacon support |
| Simple updates | Google | REST PATCH is easier than 5 web service endpoints |
| Large-scale deployments | Google | Class/Object enables bulk updates across all instances |
| Compliance/auditing | Apple | Web service approach allows detailed logging |
| Quick deployment | Google | No certificate signing infrastructure needed |
| Both platforms | Build unified model | See section 2 for field mapping tables |
