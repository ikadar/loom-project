# Domain Vocabulary – Sales & Quoting

This document defines the key domain concepts for the Sales & Quoting bounded context.

---

## Core Concepts

### Quote
A formal offer to a customer, containing line items with products and prices. Has a lifecycle: Draft → Sent → Accepted/Rejected/Expired.

### QuoteLineItem
A single line on a quote, representing a product, quantity, and price.

### Customer
An organization that can receive and accept quotes. Has contact information and addresses.

### Order
Created when a customer accepts a quote. Contains order line items derived from the quote.

### Product
An item that can be quoted and ordered. Has a price and description.

---

## Supporting Concepts

### Money
An amount with a currency (e.g., 100.00 EUR).

### Address
A postal address with street, city, postal code, country.

### ContactInfo
Email and phone number for a person.

### DateRange
A validity period with start and end dates.

---

## Relationships

- Quote **contains** 1..N QuoteLineItem
- Quote **references** Customer
- Quote **has** validity DateRange
- QuoteLineItem **references** Product
- QuoteLineItem **has** quantity and Money (unit price)
- Customer **has** Address (billing, shipping)
- Customer **has** ContactInfo
- Order **created from** Quote
- Order **contains** OrderLineItem

---

## Open Questions

- Should QuoteLineItem be tracked independently for shipping?
- Can Address change after being used on a Quote?
- Is CustomerContact a separate entity or part of Customer?
