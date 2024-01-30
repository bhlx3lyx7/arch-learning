# Checkout Blogs

[blogs](https://www.checkout.com/blog)

## Void transaction

[link](https://www.checkout.com/blog/what-is-a-void-transaction)

A void transaction is one that is cancelled by the merchant, before it’s settled with the payment service provider (PSP).
- happens after transaction is authorised, but before settled
- once the transaction is settled, it's completed, it can only be refunded
- if void a transaction, merchant won't pay any credit card processing costs, such as per-transaction fees

### scenarios
- customer request: customer changes after making purchases in short time
- incorrect transaction details: charging wrong amount, or discrepancy in customer details
- suspected fraud
- ordered goods not available: inventory not sufficient

### compare with refund
- void is much faster than refund
- void & refund happens in different phases
- void actually has no money transfer

## Authorize and capture

[link](https://www.checkout.com/blog/authorize-and-capture)

Authorization and capture are two steps in a transaction.
1. authorization checks if there's enough money
2. capture actually takes the money from the customer

### Workflow
1. issue a payment
2. request authorization: request customer's bank to set aside the funds
3. collect funds: money moves from customer to merchant

### Authorize vs. authorize & capture
Authorization confirms the customer's credit account is active, in good condition, and has enough funds for the intended transaction. If the card meets these criteria, the issuing bank temporarily sets aside the funds equal to the authorization amount.
- when authorization-only transaction is approved by the processor, the transaction is usually listed in Unsettled Transactions with a status of Authorized/Pending Capture, depending on the payment system.
- the process of authorization & capture transaction is totally automatic, it is sent to the processor for authorization and, once given the green light, it usually appears in Unsettled Transactions with the status Captured Pending Settlement.

## Payment capture

[link](https://www.checkout.com/blog/what-is-payment-capture)

Payment capture is the point at which a transaction becomes a completed payment, meaning the funds are authorized to move from the customer to the merchant.   
Capture is also the point at which fees are deducted from the pre-authorized amount. 

### Authorization vs. payment capture
Authorization is the point in the payment process that takes place immediately before capture.
- Authorization verifies the cardholder and checks with their bank that they have sufficient funds in their account to complete the transaction. If so, the bank places a hold on those funds, meaning the customers can’t spend them on anything else before capture can take place. The payment is now categorized as pending. 
- Once authorized, the merchant can submit a capture request to transfer the funds from the cardholder’s account to their account. The authorization request is usually valid for a week. If you fail to capture the customer’s funds during this time, the transaction will be canceled, and they will be free to spend the money again. 

### Instant vs. delayed payment capture
- Automatic capture is almost always instant, although you can set up an automated delay. 
- Manual capture usually requires a delay because it takes time to batch and submit multiple requests. Of course, if you have the resources, you could submit instant manual requests. 

### Delayed payment capture helps prevent chargebacks
Delaying capture is a great way to prevent chargebacks. That’s because until you submit a capture request, the payment is still pending, and the funds, while on hold, remain with the customer. 
Delayed capture could save fees for chargeback transactions, as well as the chargeback ratio, if the merchant can do more checks of customers after authorization and before capture.

## Dispute

[link](https://www.checkout.com/blog/payment-disputes)

Payment disputes are when a customer challenges a charge on their account, which can result in the issuing bank granting a chargeback. 

### Parties in dispute
- customer - who initiated the transaction and the dispute
- merchant - who accepted the transaction initially 
- payment processor - which is responsible for facilitating the transaction
- acquiring bank - which manages the merchant’s account
- issuing bank - which issued the card to the customer
- card network - which facilitates the payment

### Work flow of payment dispute
1. customer reports an issue of transaction, the bank or card issuer then notifies merchant of the disput
2. merchant investigate the transaction, collect support documentations
3. merchant respond to the dispute with evidence, to prove the transaction's validity
4. bank or card issuer reviews the evidence, makes a decision on the dispute

### Chargeback vs. dispute
- two different processes in electronic payment transactions
- dispute is often the first step of chargeback
- dispute happens from fraud, billing error, or customer dissatisfaction
- chargeback happens when issuer accepts customer's reason for dispute

## IBAN

[link](https://www.checkout.com/blog/what-is-iban)

IBAN stands for International Bank Account Number. It’s a standard international numbering system used to identify individual bank accounts in cross-border payments.

### What is IBAN
IBAN is different from your regular bank account number and sort code. Instead, it’s  used specifically for identifying your bank from overseas so you can send or receive international payments. An IBAN number starts with a two-digit country code, then two check digits, followed by up to 35 characters for the Basic Bank Account Number (BBAN). Each country chooses a BBAN format to signify its national standard for domestic payments.

IBANs are mostly reserved for European countries, but it’s different from the Single Euro Payment Area (SEPA), which is a payment network in Europe that only processes Euros.

It’s worth noting that the US and Canada don’t use the IBAN system, but they do recognize the system and process payments according to the IBAN rules.

### Example of IBAN
IBAN numbers feature a two-letter country code, two check digits, then up to 35 alphanumeric characters.

- Austria: AT 20 AT483200000012345864
- Belgium: BE 16 BE71096123456769
- Bulgaria: BG 22 BG18RZBBB91550123456789
- Kuwait: KW 81CBKU0000000000001234560101

### IBAN vs. SWIFT/BIC
- A Bank Identifier Code (BIC) identifies a bank or financial institution in an international transaction, while an IBAN shows an individual account in a specific bank in a given country.
- BIC is basically another name for a SWIFT (Society for Worldwide Interbank Financial Telecommunication) code.
- BIC gives even more detailed information to help with a transaction. It consists of a four-letter bank code, a two-letter country code, and a branch identifier, which is one letter and one number.

## PAN

[link](https://www.checkout.com/blog/primary-account-number)

A primary account number (PAN) is the series of digits – typically 16 digits, but which can vary from 12 to 19 numbers long – embossed on the front of a credit, debit, or prepaid card. The PAN is usually also encoded into the magnetic stripe on the card’s rear side.
It is a unique number assigned by the card issuer to the cardholder’s account.

### What's PAN like
- The first eight digits of the PAN are known as the Issuer Identification Number (IIN), or the Bank Identification Number (BIN).
- The latter 12 digits of the PAN (so, in a typical 16-digit pan, numbers five to 16) are the Account Identifier/Number.
- The first digit of the PAN is known as the Major Industry Identifier (MII).
- The PAN’s last number is the Validator Digit.

### How does PAN work?
1. customer initiates a card payment, PAN is collected
2. payment is tokenized, to replace PAN with a one-time unique token
3. payment is submitted to the acquiring bank, with encrypted PAN
4. issuing bank gets involved, by receiving the transaction information from acquiring bank
5. issuing bank approves the transaction, sent back to acquiring bank, and then back to seller, transaction complete

### PAN for virtual cards
PANS for virtual cards work a little differently.  
With a virtual card, the issuing bank doesn’t generate a static PAN for that card, as it would when supplying a physical card. (Static implies that the PAN doesn’t change.) Instead, tokenized virtual card transactions generate a fresh PAN every time the card is used to make a purchase – mitigating the risk of fraudulent activity.

## Request for Payment (RFP)

[link](https://www.checkout.com/blog/request-for-payments)

A Request for Payment (RFP) is any form of communication sent to a customer that both asks for and provides a way for them to complete a transaction. 

### Steps
1. customer requests a product or service from online business
2. merchant initiates the RFP to the customer via suitable or convenient method
	- the RFP message should contain identifying information for the payer and payee
3. customer reviews the payment information and complete the purchase
4. the payment is authorized and processed, the funds are transferred from the payer to the payee, both parties receive confirmation

### Ways to create RFP
- PSPs
- Email
- Mobile apps
- QR code (Quick Response code)
- Invoice software

### Benefits of using RFP
- speed
- convenience
- versatility
- control

### Best practice for creating effective RFP
- reduce steps
- automate process
- offer a variety of payment methods
- send reminders

## Accounts payable vs. accounts receivable

[link](https://www.checkout.com/blog/accounts-payable-vs-accounts-receivable)

Accounts payable and accounts receivable essentially represent money you owe and money owed to you. 

- Accounts payable is a current liability account that represents the amount you owe to your creditors and suppliers.
- Accounts receivable refers to any outstanding amounts that are owed to you by third parties.

### Differences
- Accounts payable is recorded in your general ledger as a liability, while accounts receivable is recorded as an asset
- Accounts payable represents cash flowing out of your business, whereas accounts receivable represents cash flowing into your business 

## Purchase order (PO)

[link](https://www.checkout.com/blog/purchase-order)

A purchase order is a financial document that the buyer issues to the suppliers or vendors to request their products or services. 
- PO provides details of the type and quantity of goods want to buy, as well as payment and delivery terms
- seller should be capable of fulfilling the request and the buyer is prepared to pay
- once received, PO acts as a legally binding contract between the buyer and seller

### Steps
1. buyer establishes the need and creates a purchase requisition
2. buyer creates the purchase order, including the date, names and addresses of buyer and seller, quantity of products, price, PO number (the unique id for tracking, it should be matched to the invoice)
3. supplier apprives or rejects the purchase order
4. supplier fulfills the order and the buyer pays the invoice

### Types of PO
- standard PO: one-off request for single delivery
- blanket PO: long-term agreement for recurring delivery of goods at a fixed price over a time period
- planned PO: similar to standard PO, but doesn't specify delivery date or address, e.g. restock items at irregular intervals
- contract PO: outlines the terms of agreement between buyer and seller for future orders, but not providing details for any specific order

### Purchase order vs. invoice
- purchase order is created by buyer, invoice is created by seller
- purchase order is issued to request products or services, invoice is issued to request payment for the goods
- both documents contain matching information of the terms of agreement

## Payment tokenization

[link](https://www.checkout.com/blog/payment-tokenization)

Payment tokenization is the process of replacing sensitive data in a transaction (such as the cardholder’s primary account number, or PAN) with non-sensitive data, called ‘tokens’.

These tokens don’t have any value, or significance, outside of the transaction.

### How does payment tokenization work?
1. customer initiates a transaction through merchant's online checkout page, entering the card details
2. merchant's payment gateway sends a request to the payment service provider (PSP), which tokenizes the customer's card information
3. payment service provider sends the random and unique token, which signifies the customer's card data, but has no relevance or meaning outside of the specific payments context, back to the merchant
4. merchant's payment gateway uses the token, to request authorization of the payment, to customer's bank
5. after issuing bank successfully authorizes the payment, they notify the merchant, and the payment is complete
6. merchant can store the token for future transactions from that customer, for recurring payment, refund, or enable one-click payment

### Tokenization types
- network tokenization: card information tokenized in transaction network
- PCI tokenization: tokenized card information stored in merchant's PCI storage
- digital wallet tokenization: a type of network tokenization used specifically in the case of digital wallets, the digital wallet only provides the token to the merchant

## Gross vs. net settlement

[link](https://www.checkout.com/blog/gross-vs-net-settlement)










