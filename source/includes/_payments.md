# Payments

Payments in Quaderno-lingo represent the recording of a successful payment.

Like [items](#items), payments are a sub-object and can be attached to multiple types of other records:

- Invoices and sales receipts
- Expenses

<aside class="notice">
Note that unlike items, payments cannot exist in Quaderno separately to an instance of one of these classes, and the same payment cannot be referenced by more than one instance.
</aside>

## Create a payment

> `POST /invoices/INVOICE_ID/payments.json` or `POST /expenses/EXPENSE_ID/payments.json`

```shell
# body.json
{
  "amount":"56.60",
  "payment_method":"credit_card"
}

# curl command
curl -u YOUR_API_KEY:x \
     -H 'Content-Type: application/json' \
     -X POST \
     --data-binary @body.json \
     'https://ACCOUNT_NAME.quadernoapp.com/api/invoices/INVOICE_ID/payments.json'
```

```php?start_inline=1
$payment = new QuadernoPayment(array(
                                'date' => date('2012-10-10'),
                                'payment_method' => 'credit_card'),
                                'amount' => '56.60');

$invoice->addPayment($payment);               // Return true (success) or false (error)
$invoice->save();                             // Returns true (success) or false (error)
```

```ruby
invoice = Quaderno::Invoice.find(invoice_id)
params = {
    payment_method: 'credit_card',
    amount: '56.60'
}
invoice.add_payment(params) #=> Quaderno::Payment
```

```swift?start_inline=1
let client = Quaderno.Client(/* ... */)

let params : [String: Any] = [
 "payment_method": "credit_card",
 "amount": "56.60"
]

let readInvoice = Invoice.read(INVOICE_ID)
client.request(readInvoice) { response in
  // response will contain the result of the request.
}
```

`POST`ing to `invoices/INVOICE_ID/payments.json` or `expenses/EXPENSE_ID/payments.json` will create a new payment from the parameters passed. Note that payments can only be created as an attachment to an invoice or expense.

This will return `201 Created` and the current JSON representation of the payment if the creation was a success, along with the location of the new item in the `Location` header.

### Attributes

Attribute      | Mandatory | Type/Description
---------------|-----------|-------------------------------------------------------------------------------------------------------------------------------------
amount         | **yes**   | Decimal
date           | no        | String(255 chars) Format `YYYY-MM-DD`
payment_method | **yes**   | One of the following: `credit_card`, `cash`, `wire_transfer`, `direct_debit`, `check`, `promissory_note`, `iou`, `paypal` or `other`

## Retrieve: Get all payments on an invoice or expense

```shell
curl -u YOUR_API_KEY:x \
     -X GET 'https://ACCOUNT_NAME.quadernoapp.com/api/expenses/EXPENSE_ID/payments.json'
```

```php?start_inline=1
$expense = QuadernoExpense::find('EXPENSE_ID'); // Returns a QuadernoExpense
$payments = $expense->getPayments(); // Returns an array of QuadernoPayment
```

```ruby
expense = Quaderno::Expense.find(EXPENSE_ID) #=> Quaderno::Expense
payments = expense.payments #=> an array of Quaderno::Payment
```

```swift?start_inline=1
```

`GET`ting from `/invoices/INVOICE_ID/payments.json` or `/expenses/EXPENSE_ID/payments.json` will return all the payments on the invoice or expense in question.

## Retrieve: Get a single payment on an invoice or expense

```shell
curl -u YOUR_API_KEY:x \
     -X GET 'https://ACCOUNT_NAME.quadernoapp.com/api/expenses/EXPENSE_ID/payments/PAYMENT_ID.json'
```

```php?start_inline=1
$expense = QuadernoExpense::find('EXPENSE_ID'); // Returns a QuadernoExpense
$payments = $expense->getPayments(); // Returns an array of QuadernoPayment to iterate through
```

```ruby
expense = Quaderno::Expense.find(EXPENSE_ID) #=> Quaderno::Expense
payments = expense.payments #=> an array of Quaderno::Payment to iterate through
```

`GET`ting from `/invoices/INVOICE_ID/payments/PAYMENT_ID.json` or `/expenses/EXPENSE_ID/payments/PAYMENT_ID.json` will return that payments on the invoice or expense in question, if it exists.

## Delete a payment

> `DELETE /invoices/INVOICE_ID/payments/PAYMENT_ID.json` or `DELETE /expenses/EXPENSE_ID/payments/PAYMENT_ID.json`

```shell
curl -u YOUR_API_KEY:x \
     -X DELETE 'https://ACCOUNT_NAME.quadernoapp.com/api/expenses/EXPENSE_ID/payments/PAYMENT_ID.json'
```

```ruby
invoice = Quaderno::Invoice.find(invoice_id)
invoice.remove_payment(payment_id) #=> Boolean
```

```php?start_inline=1
$expense->removePayment($payments[2]); // Return true (success) or false (error)
```

```swift?start_inline=1
```

`DELETE`ing to `/invoices/INVOICE_ID/payments/PAYMENT_ID.json` or `/expenses/EXPENSE_ID/payments/PAYMENT_ID.json` will delete the specified payment and returns `204 No Content` if successful.
