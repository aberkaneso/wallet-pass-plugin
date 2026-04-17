# Google Wallet Technical Reference

Complete technical reference for the Google Wallet API v1. This document covers all pass types, fields, objects, and patterns for generating Google Wallet passes.

## Table of Contents

1. [Architecture: Class/Object Model](#architecture-classObject-model)
2. [Pass Types: Complete Field References](#pass-types-complete-field-references)
3. [Shared Class Fields](#shared-class-fields)
4. [Shared Object Fields](#shared-object-fields)
5. [Shared Sub-Object Types](#shared-sub-object-types)
6. [Image Specifications](#image-specifications)
7. [Pass Constraints and Restrictions](#pass-constraints-and-restrictions)
8. [Key Constraints and Rules](#key-constraints-and-rules)

---

## Architecture: Class/Object Model

### Core Concepts

- **Class**: A shared template created once, referenced by many objects. Represents the pass template/design.
- **Object**: An individual pass instance for a specific user. Multiple objects can reference a single class.
- **Pattern**: Create class → Create objects referencing that class ID

### Review Status

- Classes must reach `reviewStatus: "UNDER_REVIEW"` before objects can be saved to Google Wallet
- Objects inherit styling and module configuration from their class
- Both class and object modules (text, image, links) are displayed on the pass (up to 10 each)

### Pass Type Summary

Seven pass types are supported, each with specific required fields and unique configurations:

| Pass Type | Primary Use Case |
|-----------|------------------|
| **Generic** | Flexible, custom passes (parking, memberships, insurance) |
| **Loyalty** | Points programs, rewards cards, memberships |
| **Offer** | Coupons, promotions, discounts |
| **Gift Card** | Prepaid cards, store credit |
| **Event Ticket** | Concerts, sports, conferences |
| **Flight** | Boarding passes, flight itineraries |
| **Transit** | Bus, rail, ferry, tram tickets |

---

## Pass Types: Complete Field References

### GenericClass

Universal pass template for flexible use cases.

#### Unique Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | Yes | Format: `issuerId.identifier` (alphanumeric, `.`, `_`, `-` only) |
| `issuerName` | string | Yes | Issuer/brand name |
| `classTemplateInfo` | ClassTemplateInfo | No | Template styling |
| `imageModulesData` | Image[] | No | Max 10 images |
| `textModulesData` | TextModuleData[] | No | Max 10 text blocks |
| `linksModuleData` | LinksModuleData | No | Links/URIs |
| `enableSmartTap` | boolean | No | Enable Smart Tap redaction |
| `redemptionIssuers` | string[] | No | Issuer IDs for Smart Tap |
| `multipleDevicesAndHoldersAllowedStatus` | enum | No | See Shared Class Fields |
| `viewUnlockRequirement` | enum | No | `UNLOCK_NOT_REQUIRED`, `UNLOCK_REQUIRED_TO_VIEW` |
| `callbackOptions` | CallbackOptions | No | Event callbacks |
| `securityAnimation` | SecurityAnimation | No | Animation type (FOIL_SHIMMER) |
| `appLinkData` | AppLinkData | No | Deep linking configuration |
| `valueAddedModuleData` | ValueAddedModuleData[] | No | Max 10 additional modules |
| `merchantLocations` | MerchantLocation[] | No | Max 10 locations |

#### JSON Example

```json
{
  "id": "issuerId.GENERIC_CLASS_001",
  "issuerName": "My Parking Company",
  "reviewStatus": "UNDER_REVIEW",
  "hexBackgroundColor": "#1f76d2",
  "logo": {
    "sourceUri": {
      "uri": "https://example.com/logo.png"
    },
    "contentDescription": {
      "defaultValue": "Company Logo"
    }
  },
  "textModulesData": [
    {
      "id": "header",
      "header": "Parking Pass",
      "body": "Valid in all zones"
    }
  ]
}
```

---

### GenericObject

Individual parking pass, membership, or utility pass instance.

#### Unique Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | Yes | Format: `issuerId.identifier` |
| `classId` | string | Yes | Reference to GenericClass ID |
| `cardTitle` | LocalizedString | No | Pass title |
| `subheader` | LocalizedString | No | Subheading |
| `header` | LocalizedString | No | Main header |
| `logo` | Image | No | Pass-specific logo |
| `wideLogo` | Image | No | Wide logo (2:1) |
| `genericType` | enum | No | See table below |
| `notifications` | Notification | No | Expiry/upcoming notifications |

#### genericType Enum Values


#### JSON Example

```json
{
  "id": "issuerId.GENERIC_OBJECT_001",
  "classId": "issuerId.GENERIC_CLASS_001",
  "state": "ACTIVE",
  "barcode": {
    "type": "QR_CODE",
    "value": "abc123xyz"
  },
  "cardTitle": {
    "defaultValue": "Parking Pass"
  },
  "validTimeInterval": {
    "start": {
      "date": "2026-04-08T00:00:00Z"
    },
    "end": {
      "date": "2026-12-31T23:59:59Z"
    }
  }
}
```

---

### LoyaltyClass

Rewards program, points system, or membership template.

#### Unique Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | Yes | Format: `issuerId.identifier` |
| `issuerName` | string | Yes | Issuer name |
| `programName` | string | Yes | Loyalty program name |
| `programLogo` | Image | Yes | Logo (1:1, 800x800px) |
| `accountNameLabel` | string | No | Label for account name |
| `accountIdLabel` | string | No | Label for account ID |
| `rewardsTierLabel` | string | No | Primary rewards tier label |
| `rewardsTier` | string | No | Current tier name |
| `secondaryRewardsTierLabel` | string | No | Secondary tier label |
| `secondaryRewardsTier` | string | No | Secondary tier name |
| `localizedProgramName` | LocalizedString | No | Localized program name |
| `wideProgramLogo` | Image | No | Wide logo (2:1, 1600x800px) |
| `discoverableProgram` | DiscoverableProgram | No | Sign-up configuration |

#### DiscoverableProgram Structure

```json
{
  "state": "DISCOVERABLE",
  "merchantSignupInfo": {
    "signupWebsite": "https://example.com/signup",
    "signupSharedDatas": ["FIRST_NAME", "LAST_NAME", "EMAIL"]
  },
  "merchantSigninInfo": {
    "signinWebsite": "https://example.com/signin"
  }
}
```

#### JSON Example

```json
{
  "id": "issuerId.LOYALTY_CLASS_001",
  "issuerName": "Premium Coffee Co",
  "programName": "Coffee Rewards",
  "programLogo": {
    "sourceUri": {
      "uri": "https://example.com/rewards-logo.png"
    },
    "contentDescription": {
      "defaultValue": "Rewards Logo"
    }
  },
  "reviewStatus": "UNDER_REVIEW",
  "accountNameLabel": "Member Name",
  "accountIdLabel": "Member ID",
  "rewardsTierLabel": "Status",
  "hexBackgroundColor": "#8b4513"
}
```

---

### LoyaltyObject

Individual loyalty card/membership instance.

#### Unique Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | Yes | Format: `issuerId.identifier` |
| `classId` | string | Yes | Reference to LoyaltyClass ID |
| `accountName` | string | Yes | Member name |
| `accountId` | string | Yes | Member number/ID |
| `loyaltyPoints` | LoyaltyPoints | No | Points structure (see below) |
| `secondaryLoyaltyPoints` | LoyaltyPoints | No | Secondary points (tier, miles, etc.) |
| `linkedOfferIds` | string[] | No | Associated offer IDs |

#### LoyaltyPoints Structure

Can use one of: integer, double, string, or Money object

```json
{
  "label": "Points",
  "balance": 5000
}
```

```json
{
  "label": "Miles",
  "balance": {
    "micros": 150000000,
    "currencyCode": "USD"
  }
}
```

#### JSON Example

```json
{
  "id": "issuerId.LOYALTY_OBJECT_001",
  "classId": "issuerId.LOYALTY_CLASS_001",
  "state": "ACTIVE",
  "accountName": "John Doe",
  "accountId": "MEMBER123456",
  "loyaltyPoints": {
    "label": "Points",
    "balance": 2500
  },
  "secondaryLoyaltyPoints": {
    "label": "VIP Status",
    "balance": "Gold"
  }
}
```

---

### OfferClass

Coupon, promotion, or discount template.

#### Unique Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | Yes | Format: `issuerId.identifier` |
| `issuerName` | string | Yes | Issuer name |
| `title` | string | Yes | Offer title |
| `provider` | string | Yes | Provider/merchant name |
| `titleImage` | Image | No | Main offer image |
| `details` | string | No | Detailed offer description |
| `finePrint` | string | No | Terms and conditions |
| `helpUri` | Uri | No | Help/info link |
| `localizedTitle` | LocalizedString | No | Localized title |
| `localizedDetails` | LocalizedString | No | Localized details |
| `localizedFinePrint` | LocalizedString | No | Localized T&C |
| `localizedProvider` | LocalizedString | No | Localized provider name |
| `localizedShortTitle` | LocalizedString | No | Short title (localized) |
| `shortTitle` | string | No | Short offer title |
| `redemptionChannel` | enum | No | `INSTORE`, `ONLINE`, `BOTH`, `TEMPORARY_PRICE_REDUCTION` |

#### JSON Example

```json
{
  "id": "issuerId.OFFER_CLASS_001",
  "issuerName": "RetailCo",
  "title": "20% Off Entire Purchase",
  "provider": "RetailCo Stores",
  "details": "Valid on in-store purchases over $50",
  "finePrint": "Expires 2026-12-31. Cannot be combined with other offers.",
  "redemptionChannel": "BOTH",
  "reviewStatus": "UNDER_REVIEW"
}
```

---

### OfferObject

Individual coupon/promotion instance.

#### Fields

Offers use standard shared object fields. No unique required fields beyond `id`, `classId`, and `state`.

#### JSON Example

```json
{
  "id": "issuerId.OFFER_OBJECT_001",
  "classId": "issuerId.OFFER_CLASS_001",
  "state": "ACTIVE",
  "barcode": {
    "type": "AZTEC",
    "value": "offercode123"
  },
  "validTimeInterval": {
    "start": {
      "date": "2026-04-08T00:00:00Z"
    },
    "end": {
      "date": "2026-06-30T23:59:59Z"
    }
  }
}
```

---

### GiftCardClass

Prepaid card or gift card template.

#### Unique Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | Yes | Format: `issuerId.identifier` |
| `issuerName` | string | Yes | Issuer name |
| `merchantName` | string | Yes | Merchant/brand name |
| `programLogo` | Image | No | Logo (1:1) |
| `wideProgramLogo` | Image | No | Wide logo (2:1) |
| `pinLabel` | string | No | Label for PIN field |
| `eventNumberLabel` | string | No | Label for event number |
| `cardNumberLabel` | string | No | Label for card number |
| `allowBarcodeRedemption` | boolean | No | Barcode redemption enabled |
| `localizedMerchantName` | LocalizedString | No | Localized merchant name |
| `localizedPinLabel` | LocalizedString | No | Localized PIN label |
| `localizedEventNumberLabel` | LocalizedString | No | Localized event label |
| `localizedCardNumberLabel` | LocalizedString | No | Localized card label |

#### JSON Example

```json
{
  "id": "issuerId.GIFTCARD_CLASS_001",
  "issuerName": "TechStore",
  "merchantName": "TechStore Gift Card",
  "cardNumberLabel": "Card Number",
  "pinLabel": "PIN",
  "reviewStatus": "UNDER_REVIEW",
  "hexBackgroundColor": "#ff6b35"
}
```

---

### GiftCardObject

Individual gift card instance.

#### Unique Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | Yes | Format: `issuerId.identifier` |
| `classId` | string | Yes | Reference to GiftCardClass ID |
| `cardNumber` | string | Yes | Card number (PII: may be masked) |
| `pin` | string | No | PIN code |
| `balance` | Money | No | Current balance (micros + currencyCode) |
| `balanceUpdateTime` | DateTime | No | Last balance update |
| `eventNumber` | string | No | Event/transaction number |

#### JSON Example

```json
{
  "id": "issuerId.GIFTCARD_OBJECT_001",
  "classId": "issuerId.GIFTCARD_CLASS_001",
  "state": "ACTIVE",
  "cardNumber": "****1234",
  "balance": {
    "micros": 50000000,
    "currencyCode": "USD"
  },
  "barcode": {
    "type": "CODE_128",
    "value": "gc12345678"
  }
}
```

---

### EventTicketClass

Concert, sports, or conference event template.

#### Unique Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | Yes | Format: `issuerId.identifier` |
| `issuerName` | string | Yes | Issuer name |
| `eventName` | LocalizedString | Yes | Event name |
| `eventId` | string | No | Event identifier |
| `logo` | Image | Yes | Logo (1:1, 800x800px) |
| `venue` | Venue | No | See structure below |
| `dateTime` | DateTime | No | Event timing details |
| `finePrint` | string | No | Terms and conditions |
| `confirmationCodeLabel` | LocalizedString\|enum | No | Label or `CONFIRMATION_CODE` |
| `seatLabel` | LocalizedString\|enum | No | Label or `SEAT` |
| `rowLabel` | LocalizedString\|enum | No | Label or `ROW` |
| `sectionLabel` | LocalizedString\|enum | No | Label or `SECTION` |
| `gateLabel` | LocalizedString\|enum | No | Label or `GATE` |
| `wideLogo` | Image | No | Wide logo (2:1) |
| `localizedIssuerName` | LocalizedString | No | Localized issuer |

#### Venue Structure

```json
{
  "name": {
    "defaultValue": "Madison Square Garden"
  },
  "address": {
    "defaultValue": "33 W 33rd St, New York, NY 10001"
  }
}
```

#### DateTime Structure (for Events)

```json
{
  "doorsOpen": "2026-06-15T19:00:00Z",
  "start": "2026-06-15T20:00:00Z",
  "end": "2026-06-15T23:00:00Z",
  "doorsOpenLabel": {
    "defaultValue": "Doors Open"
  }
}
```

#### JSON Example

```json
{
  "id": "issuerId.EVENT_CLASS_001",
  "issuerName": "Concerts Inc",
  "eventName": {
    "defaultValue": "Summer Music Festival 2026"
  },
  "eventId": "summer-fest-2026",
  "logo": {
    "sourceUri": {
      "uri": "https://example.com/event-logo.png"
    }
  },
  "venue": {
    "name": {
      "defaultValue": "Central Park"
    },
    "address": {
      "defaultValue": "New York, NY"
    }
  },
  "reviewStatus": "UNDER_REVIEW"
}
```

---

### EventTicketObject

Individual event ticket instance.

#### Unique Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | Yes | Format: `issuerId.identifier` |
| `classId` | string | Yes | Reference to EventTicketClass ID |
| `seatInfo` | EventSeat | No | Seating details |
| `reservationInfo` | EventReservationInfo | No | Reservation details |
| `ticketHolderName` | string | No | Name of ticket holder |
| `ticketNumber` | string | No | Unique ticket number |
| `ticketType` | LocalizedString | No | Type of ticket (VIP, Standard, etc.) |
| `faceValue` | Money | No | Ticket price |
| `linkedOfferIds` | string[] | No | Associated offers |

#### EventSeat Structure

```json
{
  "kind": "EventSeat",
  "seat": {
    "defaultValue": "A10"
  },
  "row": {
    "defaultValue": "A"
  },
  "section": {
    "defaultValue": "101"
  },
  "gate": {
    "defaultValue": "Main"
  }
}
```

#### EventReservationInfo Structure

```json
{
  "kind": "EventReservationInfo",
  "confirmationCode": "CONF123456"
}
```

#### JSON Example

```json
{
  "id": "issuerId.EVENT_OBJECT_001",
  "classId": "issuerId.EVENT_CLASS_001",
  "state": "ACTIVE",
  "ticketHolderName": "Jane Smith",
  "ticketNumber": "TKT1234567890",
  "ticketType": {
    "defaultValue": "General Admission"
  },
  "faceValue": {
    "micros": 9999000,
    "currencyCode": "USD"
  },
  "seatInfo": {
    "kind": "EventSeat",
    "seat": {
      "defaultValue": "A10"
    },
    "row": {
      "defaultValue": "A"
    },
    "section": {
      "defaultValue": "Lower"
    }
  },
  "reservationInfo": {
    "kind": "EventReservationInfo",
    "confirmationCode": "CONF987654"
  },
  "barcode": {
    "type": "QR_CODE",
    "value": "evt123456789"
  }
}
```

---

### FlightClass

Airline boarding pass template.

#### Unique Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | Yes | Format: `issuerId.identifier` |
| `issuerName` | string | Yes | Airline name |
| `flightHeader` | FlightCarrier | Yes | Carrier info |
| `flightNumber` | string | Yes | Flight number (digits only, e.g., "123") |
| `operatingCarrier` | FlightCarrier | No | Actual operating airline |
| `operatingFlightNumber` | string | No | Operating flight number |
| `flightNumberDisplayOverride` | string | No | Custom flight display |
| `origin` | AirportInfo | Yes | Departure airport |
| `destination` | AirportInfo | Yes | Arrival airport |
| `localScheduledDepartureDateTime` | string | Yes | ISO 8601 local time (no offset) |
| `localEstimatedOrActualDepartureDateTime` | string | No | Actual/estimated departure |
| `localBoardingDateTime` | string | No | Boarding time |
| `localScheduledArrivalDateTime` | string | Yes | ISO 8601 local time (no offset) |
| `localEstimatedOrActualArrivalDateTime` | string | No | Actual/estimated arrival |
| `localGateClosingDateTime` | string | No | Gate closing time |
| `flightStatus` | enum | No | `SCHEDULED`, `ACTIVE`, `LANDED`, `CANCELLED`, `REDIRECTED`, `DIVERTED` |
| `boardingAndSeatingPolicy` | BoardingPolicy | No | Boarding/seating rules |
| `languageOverride` | string | No | BCP 47 language code |

#### FlightCarrier Structure

```json
{
  "kind": "FlightCarrier",
  "carrierIataCode": "AA",
  "carrierIcaoCode": "AAL",
  "airlineName": {
    "defaultValue": "American Airlines"
  },
  "airlineLogo": {
    "sourceUri": {
      "uri": "https://example.com/aa-logo.png"
    }
  },
  "airlineAllianceLogo": {
    "sourceUri": {
      "uri": "https://example.com/alliance-logo.png"
    }
  },
  "wideAirlineLogo": {
    "sourceUri": {
      "uri": "https://example.com/aa-logo-wide.png"
    }
  }
}
```

#### AirportInfo Structure

```json
{
  "kind": "AirportInfo",
  "airportIataCode": "JFK",
  "terminal": "4",
  "gate": "B10",
  "airportNameOverride": {
    "defaultValue": "John F. Kennedy International"
  }
}
```

#### BoardingPolicy Structure

```json
{
  "boardingPolicy": "ZONE_BASED",
  "seatClassPolicy": "CABIN_BASED"
}
```

**Enum Values:**
- `boardingPolicy`: `ZONE_BASED`, `GROUP_BASED`, `BOARDING_POLICY_OTHER`
- `seatClassPolicy`: `CABIN_BASED`, `CLASS_BASED`, `TIER_BASED`, `SEAT_CLASS_POLICY_OTHER`

#### JSON Example

```json
{
  "id": "issuerId.FLIGHT_CLASS_001",
  "issuerName": "Airlines Inc",
  "flightNumber": "450",
  "flightHeader": {
    "kind": "FlightCarrier",
    "carrierIataCode": "AA",
    "airlineName": {
      "defaultValue": "American Airlines"
    }
  },
  "origin": {
    "kind": "AirportInfo",
    "airportIataCode": "JFK",
    "terminal": "4"
  },
  "destination": {
    "kind": "AirportInfo",
    "airportIataCode": "LAX",
    "terminal": "1"
  },
  "localScheduledDepartureDateTime": "2026-06-15T14:30:00",
  "localScheduledArrivalDateTime": "2026-06-15T17:30:00",
  "reviewStatus": "UNDER_REVIEW"
}
```

---

### FlightObject

Individual boarding pass instance.

#### Unique Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | Yes | Format: `issuerId.identifier` |
| `classId` | string | Yes | Reference to FlightClass ID |
| `passengerName` | string | Yes | Passenger name |
| `boardingAndSeatingInfo` | BoardingAndSeatingInfo | No | Seating/boarding position |
| `reservationInfo` | ReservationInfo | Yes | Reservation details |
| `securityProgramLogo` | Image | No | TSA PreCheck/etc logo |

#### BoardingAndSeatingInfo Structure

```json
{
  "kind": "BoardingAndSeatingInfo",
  "boardingGroup": "A",
  "seatNumber": "12A",
  "seatClass": "FIRST",
  "boardingPrivilegeImage": {
    "sourceUri": {
      "uri": "https://example.com/first-class.png"
    }
  },
  "boardingPosition": "005",
  "sequenceNumber": "0001",
  "boardingDoor": "FRONT",
  "seatAssignment": {
    "defaultValue": "12A"
  }
}
```

#### ReservationInfo Structure

```json
{
  "kind": "ReservationInfo",
  "confirmationCode": "ABCDEF",
  "eticketNumber": "0011234567890",
  "frequentFlyerInfo": {
    "frequentFlyerProgramName": {
      "defaultValue": "AAdvantage"
    },
    "frequentFlyerNumber": "AA123456789"
  }
}
```

#### JSON Example

```json
{
  "id": "issuerId.FLIGHT_OBJECT_001",
  "classId": "issuerId.FLIGHT_CLASS_001",
  "state": "ACTIVE",
  "passengerName": "John Doe",
  "boardingAndSeatingInfo": {
    "kind": "BoardingAndSeatingInfo",
    "boardingGroup": "B",
    "seatNumber": "12A",
    "seatClass": "FIRST",
    "boardingPosition": "005"
  },
  "reservationInfo": {
    "kind": "ReservationInfo",
    "confirmationCode": "ABC123",
    "eticketNumber": "0011111111111",
    "frequentFlyerInfo": {
      "frequentFlyerProgramName": {
        "defaultValue": "AAdvantage"
      },
      "frequentFlyerNumber": "AA123456789"
    }
  },
  "barcode": {
    "type": "AZTEC",
    "value": "M1ABC123DEF4567"
  }
}
```

---

### TransitClass

Public transportation template (bus, rail, ferry, tram).

#### Unique Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | Yes | Format: `issuerId.identifier` |
| `issuerName` | string | Yes | Transit operator |
| `transitOperatorName` | LocalizedString | No | Operator name |
| `logo` | Image | Yes | Logo (1:1) |
| `watermark` | Image | No | Background watermark |
| `transitType` | enum | Yes | `BUS`, `RAIL`, `TRAM`, `FERRY`, `OTHER` |
| `enableSingleLegItinerary` | boolean | No | Single leg display |
| `activationOptions` | ActivationOptions | No | Activation URL/reactivation |
| `wideLogo` | Image | No | Wide logo (2:1) |
| 19 custom labels | LocalizedString | No | See table below |

#### Transit Custom Labels

| Label Field | Purpose |
|-------------|---------|
| `customTransitTerminusNameLabel` | Terminal/station name |
| `customTicketNumberLabel` | Ticket ID |
| `customRouteRestrictionsLabel` | Route limits |
| `customRouteRestrictionsDetailsLabel` | Route detail label |
| `customTimeRestrictionsLabel` | Time limitations |
| `customOtherRestrictionsLabel` | Other limits |
| `customPurchaseReceiptNumberLabel` | Receipt ID |
| `customConfirmationCodeLabel` | Confirmation ID |
| `customPurchaseFaceValueLabel` | Ticket price |
| `customPurchasePriceLabel` | Paid amount |
| `customDiscountMessageLabel` | Discount notice |
| `customCarriageLabel` | Train car |
| `customSeatLabel` | Seat number |
| `customCoachLabel` | Coach number |
| `customPlatformLabel` | Platform |
| `customZoneLabel` | Zone |
| `customFareClassLabel` | Fare type |
| `customConcessionCategoryLabel` | Passenger type |
| `customFareNameLabel` | Fare name |

#### ActivationOptions Structure

```json
{
  "activationUrl": "https://example.com/activate",
  "allowReactivation": true
}
```

#### JSON Example

```json
{
  "id": "issuerId.TRANSIT_CLASS_001",
  "issuerName": "City Transit Authority",
  "transitOperatorName": {
    "defaultValue": "CTA"
  },
  "transitType": "RAIL",
  "logo": {
    "sourceUri": {
      "uri": "https://example.com/transit-logo.png"
    }
  },
  "customSeatLabel": {
    "defaultValue": "Seat"
  },
  "customZoneLabel": {
    "defaultValue": "Zone"
  },
  "reviewStatus": "UNDER_REVIEW"
}
```

---

### TransitObject

Individual transit ticket instance.

#### Unique Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | Yes | Format: `issuerId.identifier` |
| `classId` | string | Yes | Reference to TransitClass ID |
| `ticketNumber` | string | No | Ticket ID |
| `passengerNames` | string[] | No | Passenger name(s) |
| `passengerType` | enum | No | `SINGLE_PASSENGER`, `MULTIPLE_PASSENGERS` |
| `tripId` | string | No | Trip identifier |
| `ticketStatus` | enum | No | `USED`, `REFUNDED`, `EXCHANGED` (mutually exclusive with customTicketStatus) |
| `customTicketStatus` | LocalizedString | No | Custom status (mutually exclusive) |
| `concessionCategory` | enum | No | `ADULT`, `CHILD`, `SENIOR` (mutually exclusive with customConcessionCategory) |
| `customConcessionCategory` | LocalizedString | No | Custom category (mutually exclusive) |
| `ticketRestrictions` | TicketRestrictions | No | Route/time/other limits |
| `purchaseDetails` | PurchaseDetails | No | Purchase information |
| `ticketLeg` | TicketLeg | No | Single leg (mutually exclusive with ticketLegs) |
| `ticketLegs` | TicketLeg[] | No | Multiple legs (mutually exclusive with ticketLeg) |
| `tripType` | enum | Yes | `ONE_WAY`, `ROUND_TRIP` |
| `activationStatus` | ActivationStatus | No | Activation state |
| `deviceContext` | DeviceContext | No | Device token |
| `hexBackgroundColor` | string | No | Background color (hex) |

#### TicketRestrictions Structure

```json
{
  "routeRestrictions": {
    "defaultValue": "Green Line only"
  },
  "routeRestrictionsDetails": {
    "defaultValue": "Excludes express service"
  },
  "timeRestrictions": {
    "defaultValue": "Valid 6am-10pm"
  },
  "otherRestrictions": {
    "defaultValue": "Non-transferable"
  }
}
```

#### TicketLeg Structure

```json
{
  "originStationCode": "CHI",
  "originName": {
    "defaultValue": "Chicago Central"
  },
  "destinationStationCode": "ORD",
  "destinationName": {
    "defaultValue": "O'Hare"
  },
  "departureDateTime": "2026-06-15T14:30:00",
  "arrivalDateTime": "2026-06-15T15:15:00",
  "fareName": {
    "defaultValue": "Standard"
  },
  "carriage": "A",
  "platform": "2",
  "zone": "Zone 1",
  "ticketSeats": [
    {
      "fareClass": "ECONOMY",
      "coach": "A",
      "seat": "12",
      "seatAssignment": {
        "defaultValue": "12A"
      }
    }
  ]
}
```

#### PurchaseDetails Structure

```json
{
  "purchaseReceiptNumber": "RCP001",
  "purchaseDateTime": "2026-04-08T10:00:00Z",
  "accountId": "ACC123",
  "confirmationCode": "CONF456",
  "ticketCost": {
    "faceValue": {
      "micros": 2500000,
      "currencyCode": "USD"
    },
    "purchasePrice": {
      "micros": 2250000,
      "currencyCode": "USD"
    },
    "discountMessage": {
      "defaultValue": "10% early bird discount"
    }
  }
}
```

#### ActivationStatus Structure

```json
{
  "state": "ACTIVATED"
}
```

**State Enum Values:**

#### JSON Example

```json
{
  "id": "issuerId.TRANSIT_OBJECT_001",
  "classId": "issuerId.TRANSIT_CLASS_001",
  "state": "ACTIVE",
  "ticketNumber": "TKT123456",
  "passengerNames": ["John Doe"],
  "passengerType": "SINGLE_PASSENGER",
  "tripType": "ONE_WAY",
  "ticketStatus": "USED",
  "ticketLegs": [
    {
      "originStationCode": "CHI",
      "destinationStationCode": "ORD",
      "departureDateTime": "2026-06-15T14:30:00",
      "arrivalDateTime": "2026-06-15T15:15:00",
      "fareClass": "ECONOMY",
      "seat": "12A"
    }
  ],
  "purchaseDetails": {
    "purchasePrice": {
      "micros": 2500000,
      "currencyCode": "USD"
    }
  },
  "activationStatus": {
    "state": "ACTIVATED"
  },
  "barcode": {
    "type": "CODE_128",
    "value": "tkt123456789"
  }
}
```

---

## Shared Class Fields

All class types include these fields:

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | Yes | Format: `issuerId.identifier` |
| `issuerName` | string | Yes | Issuer/brand name |
| `reviewStatus` | enum | Yes | `DRAFT`, `UNDER_REVIEW` |
| `review` | Review | No | Review feedback from Google |
| `messages` | Message[] | No | Max 10 messages |
| `homepageUri` | Uri | No | Homepage link |
| `imageModulesData` | ImageModuleData[] | No | Max 10 images |
| `textModulesData` | TextModuleData[] | No | Max 10 text blocks |
| `linksModuleData` | LinksModuleData | No | Link collection |
| `hexBackgroundColor` | string | No | Hex color (#RRGGBB) |
| `heroImage` | Image | No | Hero image (1032x336px, 3:1) |
| `countryCode` | string | No | ISO 3166-1 alpha-2 code |
| `enableSmartTap` | boolean | No | Enable Smart Tap |
| `redemptionIssuers` | string[] | No | Issuer IDs for Smart Tap |
| `multipleDevicesAndHoldersAllowedStatus` | enum | No | See enum table below |
| `callbackOptions` | CallbackOptions | No | Callback configuration |
| `securityAnimation` | SecurityAnimation | No | Animation type |
| `viewUnlockRequirement` | enum | No | `UNLOCK_NOT_REQUIRED`, `UNLOCK_REQUIRED_TO_VIEW` |
| `classTemplateInfo` | ClassTemplateInfo | No | Template styling |
| `notifyPreference` | enum | No | `NOTIFY` (ephemeral) |
| `appLinkData` | AppLinkData | No | Deep linking |
| `valueAddedModuleData` | ValueAddedModuleData[] | No | Max 10 modules |
| `merchantLocations` | MerchantLocation[] | No | Max 10 locations |
| `localizedIssuerName` | LocalizedString | No | Localized issuer |

#### multipleDevicesAndHoldersAllowedStatus Enum


#### Deprecated Fields (do not use)


---

## Shared Object Fields

All object types include these fields:

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | Yes | Format: `issuerId.identifier` |
| `classId` | string | Yes | Parent class ID |
| `state` | enum | Yes | `ACTIVE`, `COMPLETED`, `EXPIRED`, `INACTIVE` |
| `barcode` | Barcode | No | Barcode data |
| `rotatingBarcode` | RotatingBarcode | No | Time-based barcode |
| `messages` | Message[] | No | Max 10 messages |
| `validTimeInterval` | TimeInterval | No | Start/end dates |
| `hasUsers` | boolean | No | Platform-set, read-only |
| `hasLinkedDevice` | boolean | No | Platform-set, read-only |
| `smartTapRedemptionValue` | string | No | ASCII only, Smart Tap value |
| `disableExpirationNotification` | boolean | No | Default: false |
| `imageModulesData` | ImageModuleData[] | No | Max 10 images |
| `textModulesData` | TextModuleData[] | No | Max 10 text blocks |
| `linksModuleData` | LinksModuleData | No | Links collection |
| `appLinkData` | AppLinkData | No | Deep linking |
| `heroImage` | Image | No | Hero image |
| `groupingInfo` | GroupingInfo | No | Grouping/sorting |
| `passConstraints` | PassConstraints | No | NFC/screenshot restrictions |
| `saveRestrictions` | SaveRestrictions | No | Email-based restrictions |
| `linkedObjectIds` | string[] | No | Related object IDs |
| `notifyPreference` | enum | No | `NOTIFY`, `DO_NOT_NOTIFY` (ephemeral) |
| `valueAddedModuleData` | ValueAddedModuleData[] | No | Max 10 modules |
| `merchantLocations` | MerchantLocation[] | No | Max 10 locations |

#### Deprecated Fields (do not use)


---

## Shared Sub-Object Types

### Barcode

Barcode/QR code on the pass.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `type` | enum | Yes | See enum table below |
| `value` | string | Yes | Encoded barcode data |
| `alternateText` | string | No | Fallback text |
| `showCodeText` | LocalizedString | No | Display format |

#### Type Enum


#### Example

```json
{
  "type": "QR_CODE",
  "value": "https://example.com/pass/abc123",
  "alternateText": "abc123",
  "showCodeText": {
    "defaultValue": "Scan to redeem"
  }
}
```

---

### RotatingBarcode

Time-based rotating barcode (TOTP).

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `type` | enum | Yes | Barcode type (from Barcode) |
| `valuePattern` | string | No | Pattern template |
| `totpDetails` | TotpDetails | No | TOTP algorithm config |
| `alternateText` | string | No | Fallback text |
| `initialRotatingBarcodeValues` | string[] | No | Offline fallback codes |
| `renderEncoding` | string | No | Encoding type |

#### TotpDetails Structure

```json
{
  "algorithm": "TOTP_SHA1",
  "periodMillis": 30000,
  "parameters": [
    {
      "key": "secret",
      "valueLength": 6
    }
  ]
}
```

#### Example

```json
{
  "type": "QR_CODE",
  "totpDetails": {
    "algorithm": "TOTP_SHA1",
    "periodMillis": 30000
  },
  "initialRotatingBarcodeValues": ["000001", "000002", "000003"]
}
```

---

### Image

Image reference with content description.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `sourceUri` | Uri | Yes | HTTPS image URL |
| `contentDescription` | LocalizedString | Yes | Alt text/accessibility |

#### Example

```json
{
  "sourceUri": {
    "uri": "https://example.com/images/logo.png"
  },
  "contentDescription": {
    "defaultValue": "Company Logo"
  }
}
```

---

### LocalizedString

Multi-language string support.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `defaultValue` | string | Yes | English/default value |
| `translatedValues` | TranslatedValue[] | No | Other languages |

#### TranslatedValue Structure

```json
{
  "language": "es",
  "value": "Valor"
}
```

#### Example

```json
{
  "defaultValue": "Hello",
  "translatedValues": [
    {
      "language": "es",
      "value": "Hola"
    },
    {
      "language": "fr",
      "value": "Bonjour"
    }
  ]
}
```

---

### Money

Currency amount.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `micros` | int64 | Yes | Amount in microcurrency (e.g., 150000000 = $150.00) |
| `currencyCode` | string | Yes | ISO 4217 code (e.g., "USD", "EUR") |

#### Example

```json
{
  "micros": 2995000,
  "currencyCode": "USD"
}
```

This represents $29.95. Calculate: `micros / 1,000,000 = currency`

---

### TimeInterval

Date range with start and end.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `start` | DateTime | No | Start date |
| `end` | DateTime | No | End date |

#### Example

```json
{
  "start": {
    "date": "2026-04-08T00:00:00Z"
  },
  "end": {
    "date": "2026-12-31T23:59:59Z"
  }
}
```

---

### DateTime

Date/time representation.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `date` | string | Yes | ISO 8601 format |

#### Format

Use ISO 8601 format. For flights/transit, use local airport/station time WITHOUT timezone offset:


For UTC times, use Z offset:


#### Example

```json
{
  "date": "2026-06-15T14:30:00"
}
```

---

### LatLongPoint

Geographic coordinates.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `latitude` | double | Yes | Latitude (-90 to 90) |
| `longitude` | double | Yes | Longitude (-180 to 180) |

#### Example

```json
{
  "latitude": 40.7128,
  "longitude": -74.0060
}
```

---

### PassConstraints

Restrictions on pass usage/visibility.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `nfcConstraint` | enum[] | No | NFC blocking flags |
| `screenshotEligibility` | enum | No | Screenshot permission |

#### nfcConstraint Enum


#### screenshotEligibility Enum


#### Example

```json
{
  "nfcConstraint": ["BLOCK_PAYMENT"],
  "screenshotEligibility": "INELIGIBLE"
}
```

---

### SaveRestrictions

Limits on who can save the pass.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `restrictToEmailSha256` | string | No | SHA-256 hash of email |

#### Example

```json
{
  "restrictToEmailSha256": "a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3"
}
```

Generate hash: `SHA256("user@example.com")`

---

### GroupingInfo

Grouping and sort order.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `groupingId` | string | Yes | Group identifier |
| `sortIndex` | int32 | No | Sort order (0-9999) |

#### Example

```json
{
  "groupingId": "trip-001",
  "sortIndex": 1
}
```

---

### CallbackOptions

Webhook configuration for pass events.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `url` | string | No | Callback webhook URL (HTTPS) |
| `updateRequestUrl` | string | No | Update request webhook (HTTPS) |

#### Example

```json
{
  "url": "https://example.com/webhook/save-delete",
  "updateRequestUrl": "https://example.com/webhook/update"
}
```

Google will POST to these URLs when:
- `url`: User saves/deletes pass
- `updateRequestUrl`: User requests pass update

---

### AppLinkData

Deep linking to native/web apps.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `androidAppLinkInfo` | AppLinkInfo | No | Android app link |
| `webAppLinkInfo` | AppLinkInfo | No | Web app link |

#### AppLinkInfo Structure

```json
{
  "appLogoImage": {
    "sourceUri": {
      "uri": "https://example.com/app-icon.png"
    }
  },
  "title": {
    "defaultValue": "Open App"
  },
  "description": {
    "defaultValue": "View in our app"
  },
  "appTarget": {
    "targetUri": "https://example.com/pass/abc123"
  }
}
```

#### Example

```json
{
  "androidAppLinkInfo": {
    "title": {
      "defaultValue": "View in App"
    },
    "appTarget": {
      "targetUri": "example://pass/abc123"
    }
  },
  "webAppLinkInfo": {
    "title": {
      "defaultValue": "View in Browser"
    },
    "appTarget": {
      "targetUri": "https://example.com/pass/abc123"
    }
  }
}
```

---

### LinksModuleData

Collection of links/URIs on the pass.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `uris` | Uri[] | Yes | Array of URI objects |

#### Uri Structure

```json
{
  "uri": "https://example.com/help",
  "description": "Help",
  "localizedDescription": {
    "defaultValue": "Get Help"
  },
  "id": "help-link"
}
```

#### Example

```json
{
  "uris": [
    {
      "uri": "https://example.com/support",
      "description": "Support",
      "id": "support"
    },
    {
      "uri": "https://example.com/terms",
      "description": "Terms",
      "id": "terms"
    }
  ]
}
```

---

### TextModuleData

Text block content on the pass.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | No | Module identifier |
| `header` | string | No | Heading (bold) |
| `body` | string | No | Content text |
| `localizedHeader` | LocalizedString | No | Localized heading |
| `localizedBody` | LocalizedString | No | Localized content |

#### Example

```json
{
  "id": "terms",
  "header": "Important Information",
  "body": "This pass is non-transferable and valid only when presented with valid ID.",
  "localizedBody": {
    "defaultValue": "English text",
    "translatedValues": [
      {
        "language": "es",
        "value": "Texto en español"
      }
    ]
  }
}
```

---

### ImageModuleData

Image block content on the pass.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | No | Module identifier |
| `mainImage` | Image | Yes | Image to display |

#### Example

```json
{
  "id": "hero",
  "mainImage": {
    "sourceUri": {
      "uri": "https://example.com/promo-image.jpg"
    },
    "contentDescription": {
      "defaultValue": "Summer Sale"
    }
  }
}
```

---

### Message

Notification message displayed on the pass.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | No | Message identifier |
| `header` | string | No | Subject/title |
| `body` | string | Yes | Message content |
| `messageType` | enum | No | `TEXT`, `EXPIRATION_NOTIFICATION` |
| `localizedHeader` | LocalizedString | No | Localized subject |
| `localizedBody` | LocalizedString | No | Localized content |
| `displayInterval` | TimeInterval | No | When to show |

#### Example

```json
{
  "id": "expiry-alert",
  "header": "Pass Expiring Soon",
  "body": "Your pass expires on December 31, 2026",
  "messageType": "EXPIRATION_NOTIFICATION",
  "displayInterval": {
    "start": {
      "date": "2026-12-15T00:00:00Z"
    },
    "end": {
      "date": "2026-12-31T23:59:59Z"
    }
  }
}
```

---

### SecurityAnimation

Visual animation/effect on the pass.

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `animationType` | enum | Yes | `FOIL_SHIMMER` |

#### Example

```json
{
  "animationType": "FOIL_SHIMMER"
}
```

---

### ValueAddedModuleData

Additional content module (max 10 per resource).

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `header` | LocalizedString | No | Module heading |
| `body` | LocalizedString | No | Module content |
| `image` | Image | No | Module image |
| `uri` | Uri | No | Associated link |
| `sortIndex` | int32 | No | Display order |

#### Example

```json
{
  "header": {
    "defaultValue": "Related Offers"
  },
  "body": {
    "defaultValue": "Save 10% on your next purchase"
  },
  "uri": {
    "uri": "https://example.com/offer",
    "description": "View Offer"
  },
  "sortIndex": 1
}
```

---

### MerchantLocation

Physical merchant location (max 10 per resource).

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `lat` | double | Yes | Latitude |
| `lon` | double | Yes | Longitude |

#### Example

```json
{
  "lat": 40.7128,
  "lon": -74.0060
}
```

---

### ClassTemplateInfo

Template styling configuration (optional).

Used to customize the appearance of the class template.

#### Example

```json
{
  "detailsTemplateOverride": "custom"
}
```

---

## Image Specifications

All images must be hosted on HTTPS URLs.

### Size Requirements

| Image Type | Dimensions | Aspect Ratio | Format | Notes |
|------------|-----------|--------------|--------|-------|
| **Logo** | 800x800 px | 1:1 | PNG/JPEG | Small icon |
| **Wide Logo** | 1600x800 px | 2:1 | PNG/JPEG | Horizontal banner |
| **Hero Image** | 1032x336 px | ~3:1 | PNG/JPEG | Large background |
| **Image Module** | 1860x930 px | ~2:1 | PNG/JPEG | Content image |
| **Title Image** (Offer) | 800x800 px | 1:1 | PNG/JPEG | Offer image |

### Hosting Requirements

- HTTPS only (no HTTP)
- Externally hosted (not embedded base64)
- Publicly accessible
- Consistent availability (Google caches images)

### Best Practices

- Use PNG for logos (transparency support)
- Use JPEG for photos
- Optimize for mobile (pass will be viewed on phones)
- Test on various screen densities
- Keep file sizes reasonable (<500KB per image)

---

## Pass Constraints and Restrictions

### Pass Constraints

Control NFC and screenshot behavior:

```json
{
  "passConstraints": {
    "nfcConstraint": ["BLOCK_PAYMENT"],
    "screenshotEligibility": "INELIGIBLE"
  }
}
```

#### nfcConstraint Options


#### screenshotEligibility Options


### Save Restrictions

Limit pass saving to specific email addresses by restricting to email SHA-256 hash:

```json
{
  "saveRestrictions": {
    "restrictToEmailSha256": "a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3"
  }
}
```

To calculate the hash:


---

## Key Constraints and Rules

### ID Format

All IDs must follow the format:

```
{issuerId}.{identifier}
```

Where:
- `issuerId`: Your Google Wallet issuer ID
- `identifier`: Alphanumeric, `.`, `_`, `-` only (no spaces or special characters)

**Examples:**

### Array Size Limits

| Field | Max Items |
|-------|-----------|
| `messages` | 10 |
| `valueAddedModuleData` | 10 |
| `merchantLocations` | 10 |
| `imageModulesData` | 10 |
| `textModulesData` | 10 |

### Mutual Exclusivity

These fields cannot be used together in a single resource:

| Scenario | Field A | Field B |
|----------|---------|---------|
| Transit Status | `ticketStatus` | `customTicketStatus` |
| Transit Category | `concessionCategory` | `customConcessionCategory` |
| Transit Legs | `ticketLeg` | `ticketLegs` |

### Flight Date/Time Format

Flight class dates must be ISO 8601 WITHOUT timezone offset (local airport time):


### Transit Date/Time Format

Transit class dates must be ISO 8601 WITHOUT timezone offset (local station time):


### notifyPreference is Ephemeral

The `notifyPreference` field is ephemeral and must be set on every request where notifications are desired. It doesn't persist.


### Deprecated Fields

Do not use these fields (no longer supported):


---

