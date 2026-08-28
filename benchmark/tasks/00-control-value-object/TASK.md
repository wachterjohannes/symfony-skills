Prices are being passed around as floats and it is causing rounding errors.

Add a `Money` type: an amount and a three-letter currency code. It needs to add two amounts
together and multiply an amount by a quantity. Adding two different currencies is a
programming error, not something to paper over.

Nothing else in the application uses it yet.
