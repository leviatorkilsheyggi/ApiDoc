# PaymentSets

The endpoint, gives access to the payment transmissions for all payments available to the company, grouped by days and payment types. A single paymentSet is therefore a collection of payments, on a specific day and with a specific type.
The purpose of this endpoint is to give total transparency and availablilty of all processed payments.

## DateFormat

The dates must be specified with following format `YYYY-MM-DD`. Optionally, you can specify additional timestamp `YYYY-MM-DD HH:MM:SS`.


# Get PaymentSets

To get a collection of paymentsets, use the endpoint available from an `HTTP_GET` at `https://api.farpay.io/{version}/paymentSets`.
A request without parameters will result in all available paymnetSets, which can be overwelming to retreive. To retreive smaller datasets, use the `fromDate` and `toDate` filter, that is available as parameters. 
To retreive payments for a single date, just input the same date in both `fromDate` and `toDate`.

