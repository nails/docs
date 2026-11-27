# Stripe

The `nails/driver-invoice-stripe` driver takes card payments through Stripe PaymentIntents. It is a payment driver for the [Invoice module](../README.md).

## Settings

| Setting              | Notes                                                                                                                                                                                                                                            |
|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Label                | The name of the provider, as seen by customers.                                                                                                                                                                                                  |
| Statement Descriptor | Required, maximum 22 characters. `{{INVOICE_REF}}` is replaced with the invoice reference. See [Statement descriptor](stripe.md#statement-descriptor).                                                                                           |
| Receipt email        | When enabled, `receipt_email` is sent with the charge so Stripe sends its own receipt, to the customer's billing email if they have one, otherwise their email.                                                                                  |
| API keys             | Separate publishable and secret keys for test and live, stored encrypted. The live keys are used in the production environment and the test keys everywhere else. A missing key fails the payment; the stored error is `Missing Stripe API Key.`               |

## API version

The driver pins the Stripe API version explicitly, rather than inheriting it from the `stripe/stripe-php` SDK or the Stripe account's default. The version is the `API_VERSION` constant on `Nails\Invoice\Driver\Payment\Stripe` and is sent as the `Stripe-Version` header on every request.

Upgrading `stripe/stripe-php` therefore does not change how the driver talks to Stripe. To move to a newer API version:

1. Review [Stripe's changelog](https://docs.stripe.com/changelog) for breaking changes to the resources the driver uses: PaymentIntents, Charges, Balance Transactions, Refunds, Customers and their sources, and Accounts and Country Specs.
2. Update the constant and anything the changelog requires.
3. Test in Stripe test mode: a new card, a saved card, a card that requires 3D Secure, a declined card and a refund.

Parameters removed in newer API versions are rejected with a `400`, so a check of the request against the changelog is not optional.

## Taking a payment

At checkout the driver loads Stripe.js (v3) and mounts a card Element. When the form is submitted the card is tokenised in the browser, and only the token is sent to your server.

A PaymentIntent is created and confirmed in one request. It is built from one of three things, checked in this order:

1. A saved payment source passed to the charge (see [Saved payment sources](stripe.md#saved-payment-sources)).
2. The `token` field submitted at checkout.
3. Explicit `stripe_source_id` and `stripe_customer_id` values in the payment data, for payments initiated in code.

If none are available the payment fails with `Must provide a payment source, ...`.

Other request details:

| Parameter                      | Value                                                                                                                                                                  |
|--------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `confirmation_method`          | `manual`, so the driver controls confirmation and any SCA step.                                                                                                        |
| `off_session`                  | `true` when the customer is not present, for example a recurring charge, otherwise `false`.                                                                            |
| `allowed_payment_method_types` | `['card']`. The driver only takes cards. `payment_method_types` was removed from the PaymentIntent API in `2026-09-30.endive`, and sending it returns a `400`.        |
| `capture_method`               | `automatic`. See [Capture and fees](stripe.md#capture-and-fees).                                                                                                       |
| `statement_descriptor_suffix`  | The configured descriptor, if any. See [Statement descriptor](stripe.md#statement-descriptor).                                                                         |
| `metadata`                     | The invoice ID and reference, plus any `metadata` in the payment data. Stripe's limits apply, so at most 20 keys, with names up to 40 characters and values up to 500. |

## Capture and fees

Newer API versions default `capture_method` to `automatic_async`, which captures the payment after the API response returns. A charge captured that way has no balance transaction, and so no fee, when the PaymentIntent is created. The driver records the fee as soon as the payment is created, so it sets `capture_method` to `automatic` to capture synchronously.

The fee is read from the charge's balance transaction. If it is somehow still missing, the payment is **not** failed, because the customer has already been charged and failing it would invite a second charge. Instead a fee of `0` is recorded and a line is written to the log naming the charge, so it can be reconciled.

## Statement descriptor

Stripe does not accept `statement_descriptor` for card payments. The driver sends the configured text as `statement_descriptor_suffix`, which Stripe appends to the statement descriptor prefix set on the Stripe account, so the final descriptor is the prefix plus the suffix.

Before it is sent, `{{INVOICE_REF}}` is replaced, the characters Stripe forbids (`<`, `>`, `\`, `'`, `"`, `*`) are removed, and the result is trimmed to 22 characters. If nothing is left, no suffix is sent.

## Strong Customer Authentication

If the card needs authentication (3D Secure), the PaymentIntent is created in the `requires_action` state and the driver returns an SCA response carrying the PaymentIntent ID. The invoice module then calls back into the driver, which:

- Confirms the PaymentIntent with the success URL as the `return_url` and redirects the customer to the URL Stripe provides to authenticate.
- When the customer returns and the PaymentIntent is `requires_confirmation`, confirms it again and completes the payment.
- Treats any other status as a failure.

## Refunds

Refunds are created against the original charge ID with the invoice's metadata, for the amount requested. The refund's balance transaction is expanded so its fee can be recorded; it is stored as a negative value.

## Saved payment sources

A saved payment source stores the Stripe source (or card) ID and the Stripe customer ID in the source's `data` as `source_id` and `customer_id`. When a source is created, the driver:

- Creates a Stripe customer, or loads the one given by `stripe_customer_id`.
- Attaches the source given by `stripe_source_id` to that customer.
- Records the card brand, last four digits and expiry on the source.

Updating a source updates its expiry (and name) on Stripe, and deleting a source deletes it from the Stripe customer. The driver also exposes `createCustomer()`, `getCustomer()`, `updateCustomer()` and `deleteCustomer()` for working with Stripe customers directly. `getCustomer()` treats a deleted customer as an error.

## Supported currencies

The driver asks Stripe for the supported payment currencies of the account's country and only offers itself for invoices in those currencies. The result is fetched once per request. If the lookup fails, the error is written to the log and the driver does not restrict currencies.

## Failures

When a charge fails the customer sees a generic message and the Stripe error is stored on the payment as its failure message and code, which is where to look when a payment fails with no trace in Stripe or the error handler.

| Stripe error                                | Message shown to the customer                                  |
|---------------------------------------------|----------------------------------------------------------------|
| Card declined (`CardException`)             | `The payment card was declined.` followed by Stripe's reason.  |
| Network problem (`ApiConnectionException`)  | `There was a problem connecting to the gateway, ...`           |
| Invalid request (`InvalidRequestException`) | `The gateway rejected the request, ...`                        |
| Any other Stripe API error                  | `There was a problem connecting to the gateway, ...`           |
| Anything else                               | `An error occurred while executing the request.`               |

A request Stripe rejects with a `400` never creates a payment in Stripe, so it will not appear in the Stripe dashboard's payments, only in its API logs under Developers.
