# Invoices

The invoice endpoint `https://api.farpay.io/{version}/invoices` gives you access to all your invoices. And where you can handle the invoices on the fly, see payment states, and how the invoice is processed in FarPay. Use Case scenarios are:

* List invoices with optional filters
* Single invoice (deep view)
* Create an invoice
* Update invoice
* Delete an invoice

**Remark!** that [all requests must have](All-Requests.md) an `X-API-KEY` and `Accept` mentioned in the header requests.

# Invoice paymentstates
![State diagram of the invoice](invoiceStates.png)

State | value | Brief description
------|-------|----------------
Not Paid | 100 | Initial state
Paid     | 200 | When a payment as been received and matches the invoice 
Scheduled | 300 | The customer has an agreement, and the invoice will be marked as scheduled
Pending  | 400 | The scheduled payment is now being processed
Rejected | 500 | The payment is being rejected by user, creditor or financial institution
Chargeback | 600 | The amount is charged back, as the monitary transaction is reversed by creditor or finansial institution
Payment failed | 700 | Paymnet failed of various causes such as, agreement was removed, or due to low account balance
N/A  | 1000 |  System errors or errors from the payment systems.


# Get all invoices
The endpoint is available from an `HTTP_GET` at `https://api.farpay.io/{version}/invoices`
Getting the invoices can be done by adding filters optional and independent filters. 
The filters are:
* From DueDate formated as `yyyy-MM-dd` (year-month-day)
* To DueDate formated as `yyyy-MM-dd` (year-month-day)
* PaymentStatus specified with above states, except `N/A`
 
The result is a list of invoice reference, which is a surface presentation of the invoice, its payment status and how it is being processed in FarPay. Remark that the details of the invoice, e.g. invoice lines and further deep details of the invoice is retreived from `https://api.farpay.io/{version}/invoices/{invoiceID}`

Here is an example of a single invoice reference:

````Javascript
[
  {
    "Id": 123,
    "Created": "2017-06-23T13:20:13.852Z",
    "InvoiceNumber": "string",
    "PaymentDueDate": "2017-06-23T13:20:13.852Z",
    "InvoiceAmount": 125,
    "ToBePaidAmount": 125,
    "PaymentReferenceStatus": "Ok",
    "PaymentType": "MobilePayInvoice" || "MobilePaySubscriptions" || 
                   "Betalingsservice" || "Leverandørservice" || 
                   "FI" || 
                   "Dankort" || "Visa" || "MasterCard",
    "SendStatus": "Queue",
    "ErrorDescription": ""
  }
]
````
Property | Description | Valid values
---------|-------------|--------------
Id       | FarPay unique reference to the invoice | `int`
Created  | Timestamp of when the invoice was created | `yyyy-MM-ddThh:mi:ssZ`
InvoiceNumber | The invoice number | `numeric`
InvoiceAmount | The total amount of the invoice, when it was created | `decimal`
ToBePaidAmount | The amount that is to be paid. When partial payments have occur, the reset is stated here | `numeric`
[SendStatus](invoice-send-status.md) | How will the invoice be processed regarding the communication to the customer | status 
ErrorDescription | Describe an occured error | `string`

# Single invoice
Getting the invoice gives you the relational insights to the customer, the subsequent invoicelines and payment handling. The endpoint is available from an `HTTP_GET` at `https://api.farpay.io/{version}/invoices/{invoiceID}`

Here is an example of a detailed invoice, that is due to be paid by Betalingsservice:

````JavasScript
{
  "Id": 12345678,
  "InvoiceDate": "23-06-2017",
  "InvoiceAmount": 100,
  "TaxAmount": 0,
  "ToBePaidAmount": 100,
  "Ean": null,
  "InvoiceNote": "",
  "InvoiceFooter": null,
  "InvoiceWasPaidManually": null,
  "Created": "2017-06-23T08:29:36.057",
  "InvoiceNumber": null,
  "PaymentDueDate": "2017-07-03T00:00:00",
  "Currency": "DKK",
  "Recepient": {
    "CustomerNumber": "1570",
    "Name": "Pernille Badenhofen",
    "Email": "pb@company.dk",
    "Street": null,
    "PostCode": null,
    "PoBox": null,
    "City": null,
    "Country": null
  },
  "Schedule": 0,
  "PaymentStatus": 100,
  "SendStatus": 200,
   "PaymentType": "MobilePayInvoice" || "MobilePaySubscriptions" || 
                   "Betalingsservice" || "Leverandørservice" || 
                   "FI" || 
                   "Dankort" || "Visa" || "MasterCard",
  "InvoiceStatus": "Ok",
  "PaymentRejectedBy": null,
  "InvoiceLines": [
    {
      "LineNumber": 0,
      "ProductNumber": "1001",
      "Description": "Vores fantastiske produkt",
      "BasePrice": 100,
      "Quantity": 1,
      "DiscountRate": 0,
      "DiscountedPrice": 0,
      "TaxRate": 0,
      "TaxAmount": 0,
      "Amount": 0
    }
  ],
  "TextLines": "This is a test\nAnd a new line",
  "PdfInvoice": {
    "Filename": "Invoice-1001.pdf",
    "Data": "VBERi0xLjcNCiW1tbW1DQoxIDAgb2JqDQo8PC..."
  },
  "PdfAttachments": [
    {
      "Filename": "Invoice-1001-attachment-01.pdf",
      "Data": "VBERi0xLjcNCiW1tbW1DQoxIDAgb2JqDQo8PC..."
    },
    {
      "Filename": "Invoice-1001-attachment-02.pdf",
      "Data": "VBERi0xLjcNCiW1tbW1DQoxIDAgb2JqDQo8PC..."
    }
  ],
}
````
# Insert invoice
When creating an invoice, the API facilitates two types of invoice-models. A regular invoice, with invoice data and multiple invoice lines data. A creditnote, that also has the same depth of invoice lines that can be refunded. The endpoint is available from an `HTTP_POST` at `https://api.farpay.io/{version}/invoices/`

## Ground rules for creating an invoice
There are a couple of rules, that needs attention before we go into the further details.
* The given amount should always be positive, both on the invoice and in the invoice lines.
* An invoice has the `InvoiceTypeCode`set to  `PIE`
* A creditnote has the `InvoiceTypeCode` set to `PCM`
* Card payments can be done instantly, both payments and creditnotes.
* `PdfInvoice` (optional) can hold your own PDF invoice layout and details. When absent, FarPay standard PDF invoice will be shown.
* `PdfAttachments` (optional), append PDF documents to the PdfInvoice, in order of appearence in the attachment list.
* An existing customer will only be referenced with `CustomerNumber`
* New customer can be created when not identified by the `CustomerNumber` - But it is recommended that the `Customers` `POST` endpoint is used to create new customers.

## Create invoice data

````JavasScript

{
  "InvoiceDate": "2019-08-10",
  "InvoiceAmount": 100,
  "TaxAmount": 25,
  "ToBePaidAmount": 125,
  "Ean": "EAN238273273828",
  "InvoiceNote": "This is a demo note",
  "InvoiceNumber": "OPTIONALNUMBER-123",
  "PaymentDueDate": "2019-10-27",
  "Currency": "DKK",
  "Recepient": {
    "CustomerNumber": "4434"
  },
  "InvoiceLines": [
    {
      "LineNumber": 1,
      "ProductNumber": "T1001",
      "Description": "Test product",
      "BasePrice": 50,
      "Quantity": 2,
      "UnitCode": "stk",
      "DiscountRate": 0,
      "DiscountedPrice": 0,
      "TaxRate": 25,
      "TaxAmount": 12.50,
      "Amount": 125
    }
  ],
  "TextLines": "Textline1\nTextline2 test"
}

````
## Error codes
The error codes, that apply to the endpoint `POST Invoices`:

Code         | Definition               | CTA (Call to action)
-------------|--------------------------|----------------------------------------------------
9000         | API key not set or bad   | Contact support, to get a new valid API key
10000        | Invoice not received     | Comply to the invoice format, mentioned above.
10009        | Invoice number used      | Invoice numbers are unique, and this one is already taken. Please use another one.

Instant errorcodes are elaborated in the instant details, reffered to below.  

# Insert invoice for instant payment
When an invoice model is posted into FarPay, the set amount can be withdrawn instantaneously. You will get a synchronous response, indicating if the request completed successfully or not.

Further details on instant payment and error codes are available from the [instant payment details](InvoiceInstantPayment.md)

# Update invoice
The invoice data cannot be updated, but how the invoice is treated in FarPay can be modified by sending a command to the invoice in order to e.g. re-process the invoice.
The endpoint is available from `PUT` at `https://api.farpay.io/{version}/invoices/{invoiceID}`.

Available operations are:

Operation | description
-------------|------------
Queue        | The invoice is placed in start position, and is now treated with the current settings, available from the sender's company
Sent         | Force the invoice to be treated as sent
ReadyToPrint | The invoice is forced to be set to be printed, and since the printjobs run daily, the status will change from print to sent
Error        | The error state, removes the invoice from the invoice workflow and haltes it - no further actions are executed on the invoices. Later changes can occur e.g. to set on Queue.

# Delete Invoice
The invoice can be marked as deleted by invoking endpoint with `DELETE` at `https://api.farpay.io/{version}/invoices/{invoiceID}`.
The delete action, is equivalent to updateing the invoice to `Error`

