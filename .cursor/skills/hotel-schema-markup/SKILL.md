---
name: hotel-schema-markup
description: >
  Develop, audit, and implement best-practice schema markup (structured data) for hotel and
  hospitality websites. Use this skill whenever the user is working on schema.org structured data
  for a hotel, resort, boutique property, or hospitality website — including implementing JSON-LD,
  choosing the right schema types per page, writing FAQPage markup, marking up rooms and offers,
  adding AggregateRating, preparing for Google AI Overviews and AI citation (GEO/AEO), auditing
  existing schema, or validating structured data. Trigger when the user mentions schema markup,
  structured data, rich results, JSON-LD, or hotel SEO.
---

# Hotel Schema Markup

## Overview

Schema markup (structured data) tells search engines and AI systems exactly what your hotel page means — not just what it says. Done well, it unlocks rich results in Google, improves click-through rates, and increasingly determines whether AI systems like ChatGPT, Perplexity, and Google AI Overviews include your property in their answers.

> **All JSON-LD examples in this skill are samples for reference only.** Replace placeholder values with your actual property data. Validate every implementation before publishing.

---

## Schema.org Inheritance Chain (Hotel)

Understanding the hierarchy prevents redundant or incorrect typing.

```
Place
  └── LocalBusiness
        └── LodgingBusiness
              └── Hotel
```

**What this means in practice:**

| Type | When to use explicitly |
|---|---|
| `Hotel` | Standard hotels — use this as the primary `@type` |
| `LodgingBusiness` | Non-hotel hospitality properties (retreats, homestays, boutique stays) that don't cleanly fit `Hotel`. Declare as `@type: "LodgingBusiness"` instead |
| `LocalBusiness` | Layer alongside `Hotel` via multi-type (`@type: ["Hotel", "LocalBusiness"]`) to strengthen local SEO signals — opening hours, Maps, local pack eligibility |
| `Place` | Already inherited via `Hotel`. No reason to declare explicitly |
| `Service` | **Do not use.** Hotels are `LodgingBusiness` entities, not services. Mistyping as `Service` is technically incorrect per schema.org |
| `WebSite` | Optional. Enables Sitelinks Search Box — only meaningful for properties with significant branded search volume. Skip for most independent hotels |

---

## Priority Stack (implement in this order)

| Priority | Schema Type | Page(s) | Impact |
|---|---|---|---|
| 1 | `Hotel` + `LocalBusiness` | Every property page | Foundation — `Hotel` for rich results; `LocalBusiness` for local pack + Maps |
| 2 | `FAQPage` | Property, policies, location pages | Highest AI citation uplift (~28% per 2026 data) |
| 3 | `AggregateRating` | Property page | Star ratings in SERPs |
| 4 | `HotelRoom` + `Offer` | Room/rates pages | Pricing and room details in search |
| 5 | `BreadcrumbList` | All indexable pages | Site structure signal |
| 6 | `Organization` | Homepage only | Brand entity — connects all property pages |
| 7 | `Restaurant` / `FoodEstablishment` | Dining page | Only if property has F&B |
| 8 | `Event` | Events page | Only if hotel hosts events |
| 9 | `ImageObject` | Gallery / property pages | Photo visibility in image SERPs |
| 10 | `BlogPosting` / `Article` | Blog pages | Content freshness signal |

> **Rule:** Never add schema for content that isn't visible on the page. Schema mismatching visible content is a Google guideline violation.

---

## 1. Hotel + LocalBusiness (Core Property Schema)

Required on every property page. Declare as a multi-type to get both the `Hotel` rich result eligibility and `LocalBusiness` local SEO signals in a single block.

**For standard hotels:**
```json
"@type": ["Hotel", "LocalBusiness"]
```

**For non-hotel hospitality properties** (retreats, homestays, boutique stays):
```json
"@type": ["LodgingBusiness", "LocalBusiness"]
```

Full example:

```json
{
  "@context": "https://schema.org",
  "@type": ["Hotel", "LocalBusiness"],
  "name": "The Harbour View Hotel",
  "url": "https://www.example-hotel.com",
  "description": "A boutique seafront hotel with 42 rooms, rooftop pool, and direct beach access.",
  "telephone": "+91-484-000-0000",
  "email": "reservations@example-hotel.com",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "12 Marine Drive",
    "addressLocality": "Kochi",
    "addressRegion": "Kerala",
    "postalCode": "682001",
    "addressCountry": "IN"
  },
  "geo": {
    "@type": "GeoCoordinates",
    "latitude": 9.9312,
    "longitude": 76.2673
  },
  "starRating": {
    "@type": "Rating",
    "ratingValue": "4"
  },
  "priceRange": "₹5000 - ₹15000",
  "numberOfRooms": 42,
  "checkinTime": "14:00",
  "checkoutTime": "12:00",
  "amenityFeature": [
    { "@type": "LocationFeatureSpecification", "name": "Free WiFi", "value": true },
    { "@type": "LocationFeatureSpecification", "name": "Swimming Pool", "value": true },
    { "@type": "LocationFeatureSpecification", "name": "Airport Shuttle", "value": true },
    { "@type": "LocationFeatureSpecification", "name": "Breakfast Included", "value": false }
  ],
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": 4.4,
    "reviewCount": 312,
    "bestRating": 5,
    "worstRating": 1
  },
  "image": [
    "https://www.example-hotel.com/images/facade.jpg",
    "https://www.example-hotel.com/images/room-deluxe.jpg"
  ],
  "sameAs": [
    "https://www.tripadvisor.com/Hotel-your-property-url",
    "https://www.booking.com/hotel/your-property-url"
  ],
  "openingHoursSpecification": {
    "@type": "OpeningHoursSpecification",
    "dayOfWeek": [
      "Monday","Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday"
    ],
    "opens": "00:00",
    "closes": "23:59"
  }
}
```

**Critical fields — never omit:**
- `name`, `address` (with `postalCode` + `addressCountry`), `telephone`, `geo`
- `starRating` — use this for official star classifications, NOT for guest ratings (use `aggregateRating` for that)
- `aggregateRating` — include both `ratingValue` AND `reviewCount`. A rating without a count is untrustworthy to AI systems.
- `sameAs` — links your entity to OTA profiles; helps AI systems confirm your property identity

---

## 2. FAQPage

Highest-impact schema for AI citation. Add to property pages, location pages, and policy pages. Target 8–12 questions per page. Write answers as you'd give them to a guest — specific and complete.

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What time is check-in and check-out?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Check-in is from 2:00 PM. Check-out is by 12:00 PM. Early check-in and late check-out may be arranged subject to availability — please contact the front desk in advance."
      }
    },
    {
      "@type": "Question",
      "name": "Is parking available at the hotel?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Yes. Complimentary self-parking is available for hotel guests. Valet parking is available at ₹300 per day."
      }
    },
    {
      "@type": "Question",
      "name": "Is the hotel pet-friendly?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "We welcome small pets (under 10 kg) in designated pet-friendly rooms. A ₹500 per night pet fee applies. Please inform us at booking."
      }
    },
    {
      "@type": "Question",
      "name": "How far is the hotel from the airport?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "The hotel is 28 km from Cochin International Airport, approximately 45 minutes by car. We offer a paid airport transfer service — book at least 24 hours in advance."
      }
    }
  ]
}
```

**Good FAQ topics for hotels:** check-in/out times, parking, transport links, cancellation policy, pets, accessibility, dining hours, room types, pool access, Wi-Fi, breakfast.

> **Note (2026):** Google no longer shows FAQ rich result dropdowns for most commercial pages. However, FAQPage schema still meaningfully improves AI citation probability and content understanding signals. Do not skip it.

---

## 3. HotelRoom + Offer (Room Pages)

Each room category should have its own page and its own schema block. Room is typed as both `HotelRoom` and `Product` (Multi-Typed Entity pattern required by schema.org).

```json
{
  "@context": "https://schema.org",
  "@type": ["HotelRoom", "Product"],
  "name": "Deluxe Sea View Room",
  "description": "A 42 sqm room with floor-to-ceiling windows and direct ocean views. King bed, rainfall shower, complimentary minibar.",
  "photo": "https://www.example-hotel.com/images/deluxe-sea-view.jpg",
  "bed": {
    "@type": "BedDetails",
    "typeOfBed": "King",
    "numberOfBeds": 1
  },
  "occupancy": {
    "@type": "QuantitativeValue",
    "minValue": 1,
    "maxValue": 2
  },
  "amenityFeature": [
    { "@type": "LocationFeatureSpecification", "name": "Ocean View", "value": true },
    { "@type": "LocationFeatureSpecification", "name": "Air Conditioning", "value": true },
    { "@type": "LocationFeatureSpecification", "name": "Minibar", "value": true }
  ],
  "containedInPlace": {
    "@type": "Hotel",
    "name": "The Harbour View Hotel",
    "url": "https://www.example-hotel.com"
  },
  "offers": {
    "@type": "Offer",
    "price": "8500",
    "priceCurrency": "INR",
    "priceValidUntil": "2026-12-31",
    "availability": "https://schema.org/InStock",
    "url": "https://www.example-hotel.com/rooms/deluxe-sea-view",
    "businessFunction": "http://purl.org/goodrelations/v1#LeaseOut"
  }
}
```

> `businessFunction: LeaseOut` is required by schema.org for rental (not sale) of accommodation. Do not omit.

---

## 4. Organization (Homepage Only)

Establish the brand entity on the homepage. For single properties, this wraps around the Hotel entity. For hotel groups, use `Organization` on the homepage and individual `Hotel` blocks on each property page.

```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Harbour View Hotels",
  "url": "https://www.example-hotel.com",
  "logo": {
    "@type": "ImageObject",
    "url": "https://www.example-hotel.com/images/logo.png",
    "width": 300,
    "height": 100
  },
  "sameAs": [
    "https://www.instagram.com/example-hotel",
    "https://www.facebook.com/example-hotel",
    "https://www.linkedin.com/company/example-hotel"
  ]
}
```

> Keep the logo URL stable. AI systems use this as an identity anchor across citations.

---

## 6a. WebSite (Optional — High-Traffic Brands Only)

Enables the Sitelinks Search Box in Google. Only worth implementing if your hotel has significant branded search volume (guests actively searching your property name). Skip for most independent properties.

```json
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "The Harbour View Hotel",
  "url": "https://www.example-hotel.com",
  "potentialAction": {
    "@type": "SearchAction",
    "target": {
      "@type": "EntryPoint",
      "urlTemplate": "https://www.example-hotel.com/search?q={search_term_string}"
    },
    "query-input": "required name=search_term_string"
  }
}
```

> Only add this if your site has a functioning search feature. Adding it without a working search endpoint is a guideline violation.

---

## 5. BreadcrumbList (All Indexable Pages)

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://www.example-hotel.com/" },
    { "@type": "ListItem", "position": 2, "name": "Rooms", "item": "https://www.example-hotel.com/rooms/" },
    { "@type": "ListItem", "position": 3, "name": "Deluxe Sea View Room", "item": "https://www.example-hotel.com/rooms/deluxe-sea-view/" }
  ]
}
```

---

## 6. Event (Hotel Events Page)

Only use when the hotel hosts publicly bookable events. Do not use for internal functions.

```json
{
  "@context": "https://schema.org",
  "@type": "Event",
  "name": "Sunset Rooftop Brunch",
  "startDate": "2026-06-07T11:00:00+05:30",
  "endDate": "2026-06-07T14:00:00+05:30",
  "eventStatus": "https://schema.org/EventScheduled",
  "eventAttendanceMode": "https://schema.org/OfflineEventAttendanceMode",
  "location": {
    "@type": "Place",
    "name": "Rooftop Terrace — The Harbour View Hotel",
    "address": {
      "@type": "PostalAddress",
      "streetAddress": "12 Marine Drive",
      "addressLocality": "Kochi",
      "addressCountry": "IN"
    }
  },
  "offers": {
    "@type": "Offer",
    "price": "1500",
    "priceCurrency": "INR",
    "url": "https://www.example-hotel.com/events/sunset-brunch",
    "availability": "https://schema.org/InStock"
  },
  "organizer": {
    "@type": "Organization",
    "name": "The Harbour View Hotel",
    "url": "https://www.example-hotel.com"
  }
}
```

---

## 7. Multiple Schema Blocks on One Page

Use either multiple `<script type="application/ld+json">` tags or a single `@graph` array. `@graph` is cleaner for complex property pages.

```json
{
  "@context": "https://schema.org",
  "@graph": [
    { "@type": "Hotel", "name": "...", "..." : "..." },
    { "@type": "FAQPage", "mainEntity": [ "..." ] },
    { "@type": "BreadcrumbList", "itemListElement": [ "..." ] }
  ]
}
```

---

## 8. Page-Level Schema Map

| Page | Schema Types to Use |
|---|---|
| Homepage | `Hotel` + `LocalBusiness` + `Organization` + `BreadcrumbList` |
| Room page | `HotelRoom` + `Product` + `Offer` + `BreadcrumbList` |
| Location / about | `Hotel` + `LocalBusiness` + `FAQPage` + `BreadcrumbList` |
| Dining page | `Restaurant` (or `FoodEstablishment`) + `BreadcrumbList` |
| Events page | `Event` (one block per event) + `BreadcrumbList` |
| Blog post | `BlogPosting` + `BreadcrumbList` |
| Policies / FAQ page | `FAQPage` + `BreadcrumbList` |
| Non-hotel property (retreat, homestay) | `LodgingBusiness` + `LocalBusiness` + `FAQPage` + `BreadcrumbList` |

---

## 9. Common Mistakes

| Mistake | Fix |
|---|---|
| `starRating` used for guest ratings | Use `starRating` for official star classification only; use `aggregateRating` for guest scores |
| `aggregateRating` without `reviewCount` | Always pair `ratingValue` + `reviewCount` + `bestRating` + `worstRating` |
| Hotel group uses one `Hotel` schema on homepage | Each property needs its own schema on its own page; homepage uses `Organization` |
| Schema on page with no matching visible content | Violation of Google guidelines — only mark up what's on the page |
| `FAQPage` with 2–3 questions | Aim for 8–12; sparse FAQ schema has minimal AI citation impact |
| `sameAs` omitted | Include OTA profile URLs; critical for AI entity resolution |
| `priceRange` or `offers` never updated | Stale pricing in schema degrades trust signals — update quarterly |
| Missing `postalCode` + `addressCountry` in `PostalAddress` | Required for geo verification by AI systems |
| Room offers missing `businessFunction: LeaseOut` | Required per schema.org spec for rental accommodations |

---

## 10. Validation & Monitoring

**Before publishing:**
- Google Rich Results Test: https://search.google.com/test/rich-results
- Schema.org Validator: https://validator.schema.org

**After publishing:**
- Google Search Console → Enhancements tab → check for errors on Hotel, FAQ, Breadcrumb
- Update `aggregateRating` at minimum quarterly
- Test AI citation: ask ChatGPT, Perplexity, and Google AI Overview — "best [category] hotels in [city]" — and check if your property appears and is described accurately

**WordPress plugins (if applicable):**
- Rank Math — covers Hotel, FAQ, BreadcrumbList
- Schema Pro — supports `HotelRoom` + `Offer`
- For custom JSON-LD beyond plugin support, inject via `wp_head` hook or a custom block

---

## 11. AI Discoverability Notes (GEO/AEO)

AI systems (ChatGPT, Perplexity, Claude, Google AI Overviews) primarily read your HTML content during live retrieval, but use schema from indexed pages to build their property understanding between queries. Schema shapes what AI "knows" about your hotel before a query arrives.

Key signals AI systems use to cite a hotel:
- `aggregateRating` with `reviewCount` (confidence signal)
- `FAQPage` (pre-formatted answers — highest extraction efficiency)
- `geo` coordinates (proximity queries: "hotels near X")
- `amenityFeature` list (feature-match queries: "hotels with pool and beach access")
- `sameAs` OTA links (identity verification across sources)
- `description` field quality (used verbatim or paraphrased in AI answers)

Write your `description` field as a single-sentence property summary you'd be happy seeing quoted in an AI answer.
