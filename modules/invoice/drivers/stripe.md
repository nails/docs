# Stripe

The `nails/driver-invoice-stripe` driver takes card payments through Stripe. It is a payment driver for the [Invoice module](../README.md).

## Settings

| Setting              | Notes                                                                                                                                                                                             |
|----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Label                | The name of the provider, as seen by customers.                                                                                                                                                   |
| Statement Descriptor | Up to 22 characters. `{{INVOICE_REF}}` is replaced with the invoice reference. See [Statement descriptor](#statement-descriptor).                                                                 |
| Receipt email        | When enabled, Stripe sends its own receipt to the customer's billing email, or to their email if they have no billing email.                                                                      |
| API keys             | Separate publishable and secret keys for test and live. Live keys are used in production, and test keys everywhere else.                                                                         |

## Taking a payment

At checkout, the card details are entered into Stripe's card element and tokenised in the browser, so card numbers never reach your server. The driver then charges the card straight away.

Only card payments are supported. Payments are captured immediately, so the Stripe fee is recorded at the time of payment.

If a payment can't be taken, the customer sees a message explaining why where Stripe provides one, and the payment is recorded as failed. The Stripe error is stored with the payment, so check there first when investigating a failure.

## Strong Customer Authentication

If the card needs authentication (3D Secure), the customer is redirected to their card issuer to complete it, then returned to your success URL. Payments that don't need authentication complete without a redirect.

## Statement descriptor

The statement descriptor is the text shown on the customer's card statement. Stripe appends it to the prefix set on your Stripe account, so the full descriptor is the account prefix followed by the configured text.

Before it is sent, `{{INVOICE_REF}}` is replaced, characters Stripe does not allow (`<`, `>`, `\`, `'`, `"`, `*`) are removed, and the result is trimmed to 22 characters.

## Refunds

Refunds are issued against the original charge for the amount requested. The invoice reference is recorded against the refund in Stripe.

## Saved payment sources

Card details saved to an account are stored as a Stripe customer and card. The brand, last four digits and expiry are kept on the source so it can be shown to the customer. Deleting a saved source also removes it from the Stripe customer.

## Supported currencies

The driver only offers Stripe for invoices in a currency the Stripe account supports. If that list can't be fetched, the error is logged and the driver doesn't restrict currencies.

## Fees

If the fee for a payment can't be read straight after it is taken, the payment is still recorded as complete and the fee is recorded as `0`. A line is written to the log naming the charge, so it can be reconciled in Stripe.
