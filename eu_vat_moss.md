# EU VAT MOSS Compliance for Stripe

In order to meet EU VAT MOSS compliance you can use the Quaderno Transactions Flow.

With this flow you can make live taxes calculations in your website and check if the customer data meets the VAT compliance before completing the order. 

**Note: If you deal with Stripe Subscriptions, the easiest way to comply with the EU VAT MOSS rules is using [quaderno.js](https://github.com/quaderno/quaderno.js).**

## The Flow

Let's start with a practical example. Suppose your business is based in Barcelona and your customer is a German guy based in Berlin.

### Step 1: Calculate Taxes
While your customer is filling in your checkout form (card number, name, etc.), you can make live taxes calculations by calling our taxes calculate method (see the [taxes section](https://github.com/quaderno/quaderno-api/blob/master/sections/taxes.md) to get more info). 

```sh
$ curl https://myshop.quadernoapp.com/api/v1/taxes/calculate.json?country=DE&postal_code=10245&vat_number=DE345789003 \
    -u HPx1vDBKCG85X1HppFo8:x
```

This call will return a JSON object with the tax you may apply to this customer. For example:

```json
{
    "name":"VAT",
    "rate":19.0,
    "notes":null
}
```

You can use the tax rate to show the final amount your customer is going to pay.

In case you want to do this calculations via ajax, note that as it executes your code on the client side it will expose your API token. We recommend using mechanisms to hide it such as proxy pages.

### Step 2: Process and Complete the Order
Now that everything is verified and the information is stored, you can complete the order and take the payment on Stripe. First, create a Stripe customer

```php
//create a customer
Stripe_Customer::create(array(
  "description" => "Customer Full Name",
  "email" => "text@example.com",
  "card" => "tok_103NCO2eZvKYlo2Cc4lerc1E", // obtained with Stripe.js
  "metadata" => array(
    "first_name" => "First name",  // not if company
    "last_name" => "Last name",  // not if company
    "contact_person" => "", //if company
    "street_line_1" => "Boxhaneger Platz 1",
    "city" => "Berlin",
    "postal_code" => "10245",
    "region" => "Berlin",
    "country" => "DE", // code ISO 3166-1 alpha-2
    "tax_id" => "DE345789003"
  ) // all metadata are optional
));
```

Then, you can create a charge or a subscription on Stripe. If you create a charge, don't forget to send the tax data you want to apply and the customer IP address. We'll use the latter to check the customer location.

```php
//charge the customer
Stripe_Charge::create(array(
  "amount" => 1428, // final amount, including taxes
  "currency" => "eur",
  "customer" => "cus_4BnckL2duT7i4y", 
  "description" => "The Neverending Story, Michael Ende (EPUB)",
  "metadata" => array(
    "tax_name" => "VAT",
    "tax_rate" => 19,
    "ip_address" => "0.0.0.0"
  ) // all metadata are optional
));
```

If you want to create a subscription, you have to create first a Stripe Invoice Item to add the taxes to the final invoice. Don't forget to send the customer IP address if you want we to check the customer location.  

```php
$cu = Stripe_Customer::retrieve("cus_4BnckL2duT7i4y");

Stripe_InvoiceItem::create(array(
    "customer" => "cus_4BnckL2duT7i4y",
    "amount" => 119, // the tax amount in cents for a 10€ plan
    "currency" => "eur",
    "description" => "Taxes",
    "metadata" => array(
        "type" => "tax",
        "name" => "VAT",
        "rate" => 19,
    )
));

$cu->subscriptions->create(array(
    "plan" => "awesome",
    "metadata" => array(
       "ip_address" => '0.0.0.0'
    )
));
```

In both cases, Quaderno will always create an invoice and send you a notification if it cannot check the customer location using the billing country, the IP address, and the credit card country. 