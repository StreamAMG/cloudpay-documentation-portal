---
stoplight-id: fyynuq0d4trqy
---

# API-Integration-Guide

## Introduction

The CloudPay API allows you to integrate a custom purchase flow for a Sport OTT service, which includes:

- **Identity management and starting a session with CloudPay** includes customer registration, login, and forget password functionality all handle by AWS Cognito and the use of JWT token to authorise access to CloudPay
- C**heckout** that allows your customers to seamlessly purchase and consume the content you offer through subscriptions using entitlements
- **User management** that allows a customer to manage their account details

## Identity Management **and starting a session with CloudPay**

### Authentication

Authentication is handled via a Single Sign On (SSO) provider.  CloudPay has existing support for a number of OAuth providers and is configured via the CloudPay backend admin console.

> CloudPay uses AWS Cognito as its default SSO provider where a client does not have an existing service

This service will cover customer registration, login, and forget password functionality.

### Authorisation

CloudPay requires SSO clients to generate a JWT token when a user is successfully authenticated.  This JWT token is used to authorise access to the CloudPay APIs.

In order to enable a JWT SSO integration, a StreamAMG admin must enable this on your account.  For this we need the following details related to your SSO provider:

- Choice of Algorithm
- Sharing of Secrets
- Claims Map (Unique Identifier, First Name, Last Name, Email Address)

The first part of the integration relies on the client to authorise the user and generate the JWT Token on their hosted applications and authentication servers.

### Start a CloudPay session

The **Session Start API** endpoint exposes the functionality for a user to begin a CloudPay session and thus count towards the device lock total - concurrent devices logged into a single account.

This endpoint will only allow a user with a valid JWT session token to start a CloudPay session.  The JWT token should be passed as a query param as follows.

```json
?**apijwttoken**=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Once a CloudPay session has been established the session token (JWT) can be used throughout the CloudPay flow to provide trust when requesting access to other API functionality.

> The CloudPay Session has the same signature as the Third-Party JWT token

The CloudPay SSO session start counts towards the device lock which is configured via the CloudPay admin backend.  This is how you control the number of concurrent users accessing a single account. The default concurrency is 3.

### Validate an active CloudPay session

The **Prescence API** endpoint exposes the functionality to validate the activeness of a users session and whether they have any active subscriptions.

This endpoint will only allow a user with a valid session token to return the status of their session.

> Having an active session will only signal a user is authenticated and therefore logged in

CustomerEntitlements and CustomerSubscriptionCount could be used to return premium access status.  However they should be used with caution if the service offers multiple entitlements that provide access to different content types.  For example having a CustomerSubscriptionCount = 1 does not mean I have the correct entitlement to access all content where the service is using different content entitlements.

CustomerPackages can be used to return the package UUID that a user has subscribed to.  This may be used to restrict the user purchasing the same package multiple times.

## Checkout

### Return subscription packages

The **Package API** endpoint exposes the functionality to return a list of packages available to the user.

Package geo-restrictions are configured via the CloudPay admin backend with the editor deciding which countries are allowed to access which package.  When a request is made to the package API it is the ‘location’ request header that is used to determine the users location and therefore access to available packages.

This endpoint does not require a valid session token to return the available packages.  This is a deliberate decision so a user is able to see the available subscriptions without having to register for the service.

This is the first step of the checkout flow which differs for Web and Mobile integrations.

### Web integration

Each subscription plan object in the package API response contains a subscription Id.  This value is needed for the checkout API in order to pull back the amount and currency that is passed to Stripe when building the basket.

```json
"SubscriptionPlanOptions": [
{
	"Id": "720e8m37-8wv7-8621-1t18-560s5641fj9d",
```

### Mobile integration

For a mobile integration a query param should be used to return the IAP related subscriptions.  

```html
https://payments.streamamg.com/api/v1/package**?type=iap**
```

> By default the Package API endpoint will not return IAP related subscriptions


The response contains the App ProductID that should be used to return the In App Payment (IAP) subscriptions from the various app stores calls.

```json
"IAPData": [{
		"Platform": "Amazon",
		"ProductID": "AmazonClientSubscriptionID"
	},
	{
		"Platform": "Roku",
		"ProductID": "RokuClientSubscriptionID"
	},
	{
		"Platform": "Google",
		"ProductID": "GoogleClientSubscriptionID"
	},
	{
		"Platform": "Apple",
		"ProductID": "AppleClientSubscriptionID"
	}
]
```

Once the IAP subscriptions have been returned you checkout via the IAP provider payment method.  Skip to the Verify an IAP purchase for the next step.

### Purchase a Web subscription

The **Checkout API** endpoint exposes the functionality for a user to purchase a subscription plan using the SubscriptionPlanOptions ID from the Package API (see web integration).

This endpoint will only allow a user with a valid session token to purchase a subscription. 

To initiate a payment with Stripe you need to create a Payment/Setup Intent which is created via the **Checkout API.** 

In order to do this you must pass in the Package ID (SubscriptionPlanOptions ID) to the body of the request, which was obtained from the Package API.

> A Payment Intent represents the payment process for a single payment attempt.  A Setup Intent, on the other hand, is used to set up and save a customer's payment details for future use such as after a free trial or update to billing details.  The logic to decide which intent to use is handled by the Checkout API based on the package type - recurring or non-recurring

The response from the checkout API contains the basket ID (Payment or Setup Intent) and a client secret.

You use the client secret that is returned to load the Stripes UI libraries including the Stripe Payment Element (UI component to take billing details). Refer to [Stripe documentation](https://stripe.com/docs/payments/accept-a-payment?ui=elements) for more information.

Payment methods displayed via the stripe payment element are configured manually via the Stripe dashboard and are performed by StreamAMG admins.

> Not all payment methods are supported for both recurring and non-recurring subscriptions types.  For example some only support one off payments not recurring

### Apply a discount

The **Discount API** endpoint exposes the functionality for a user to enter a discount code to reduce the amount owed.

This endpoint will only allow a user with a valid session token to apply a discount code. 

To apply a discount to the checkout amount you should pass in the basket ID (Payment Intent) and the discount code to the body of the API request.

```json
{
  "basketId": "pi_1Gszkg7eZvGYlo3C6ZLlAL7t",
  "code": "50OFF"
}
```

The discounts are configured in the CloudPay Admin backend as single or multi-use codes with variable discount values.

> A discount of 100% does not record a transaction record in the database but will create a subscription and entitlement for a user

The amount is updated in the basket (Payment Intent) and the client secret is used to complete the payment process via the Stripe payment element UI where the user enters their preferred payment method details.  

> You cannot use a discount on a package with a free trial as this is using the Setup Intent that takes a payment at a later date.

### Verify an IAP purchase

The **Verify API** endpoint exposes the functionality for CloudPay to validate the in-app purchase with the provider, such as, Apple, Android, Roku and Amazon so that we can ascertain legitimacy of the purchase and avoid purchase frauds.

This endpoint will only allow a user with a valid session token to validate their in-app purchase.

Once a user has made an IAP a receipt is sent to the device which needs to be forwarded to the CloudPay Verify API endpoint.  CloudPay forwards the receipt to payment provider and on successful response a subscription and entitlement is granted to the user via CloudPay allowing them to consume premium content.

Status updates and changes to an IAP subscription are managed via two routes.

- Receipt verification
- Server notifications

Status changes include, renewals, refunds, cancellations (soft and hard) and dunning periods where the provider tries to take a renewal payment.

> If you want transaction values to appear in CloudPay for IAP subscriptions then you must send the payment object in the body of the Verify API request.  More details of what is required can be found in the stoplight documentation

## User Management

### Return My Account details

The **Account API** endpoint exposes the functionality for a user to return user, subscription and billing details.

This endpoint will only allow a user with a valid session token to return their account information.

There are additional endpoints available to make changes to the account summary or view additional account information.

- User updates - PATCH request to account API
- Billing updates - Billing update API
- Cancel subscriptions - Cancellation API
- Transaction details - Transaction API

### Update My Account details

The **Account API** also allows you to make a **PATCH** request to update the existing user details.  

Updates to the core information; Email, FirstName and LastName are done by entering a new Key:Value pair.  The custom fields are referenced and updated using the UUID that is assigned to them by CloudPay.  The custom field UUID’s are returned in the account API responses for reference when updating.

> Billing details are updated via the Update Billing API

### Update billing details

The **Billing Update API** endpoint exposes the functionality for a user to update their stripe billing details by creating a new Stripe Setup Intent that will be used when the next transaction attempt occurs.

This endpoint will only allow a user with a valid session token to update their preferred payment method / billing details.

The request will configure a Stripe Setup Intent (just like for free trial plans) and return a Client Secret that is used to load the Stripes UI libraries including the Stripe Payment Element.

The user will then update the new payment method / billing details via the Stripe payment element UI which will be used when the next payment request is made.

Once submitted this will be updated both in Stripe and as part of the billing profile returned via the account API.

### Return transaction details

The **Transaction API** endpoint exposes the functionality for a user to return a list of transactions that includes the initial transaction, renewals, refunds and failed transactions.  

This endpoint will only allow a user with a valid session token to return their transaction details.

The status value shows if a transaction has been successful or not and the ProviderMessage provides details around the status.

```json
"Status": -1,
"ProviderMessage": "Your card has insufficient funds."
```

```json
"Status": 1,
"ProviderMessage": null
```

### Cancel a subscription

The **Cancellation API** endpoint exposes the functionality for a user to cancel a subscription plan using the customers subscription identifier.

This endpoint will only allow a user with a valid session token to cancel their own subscriptions.

> The customer subscription identifier is not the subscription plan ID.  This is a unique ID that is assigned to the user for the duration of their subscription.  A user may subscribe to the same package at a later date and they will be assigned a new customer subscription identifier for the duration of that subscription.  This allows us to track relative lengths of a customers subscription from start to end.

The ID can be returned using the Account API and is found in the ‘Subscriptions’ object

```json
"Subscriptions": [
        {
            "Id": "f8a32af7-5h37-11d1-m97c-0174fe380e9a",
```

### Terminate a session with CloudPay

The **Terminate API** endpoint exposes the functionality for a user to terminate a CloudPay session once a user has finished making requests against the CloudPay API as part of the logout flow.  

This endpoint will only allow a user with a valid session token to terminate the session.

An active session counts towards the device lock number so it is important that a user session is terminated.