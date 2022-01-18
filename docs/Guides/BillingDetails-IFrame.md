# User's account Card Details update

CloudPay provides clients with hosted webpages to expose an embeddable page that contains the relevant API calls with CloudPay and Stripe to manage the card details update.

## How it works

Prerequisites:

- A client with Third-Party SSO using JWT
- A valid user JWT token

## Implementations

1. Add this code to existed html 
2. replaced **{{cloudpay-instance}}** and **{{apiJwtToken}}** with correct values

```
<iframe style="width: 100%; height: 100%; border: 0" title="User Account Card Details" sandbox="allow-scripts allow-same-origin allow-top-navigation" src="{{cloudpay-instance}}/account/v2/billing?apijwttoken={{apiJwtToken}}" />
```
Examples:

```
<iframe style="width: 100%; height: 100%; border: 0" sandbox="allow-scripts allow-same-origin allow-top-navigation" src="https://payments.streamamg.com/account/v2/billing?apijwttoken=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c" />
```

## How to:
- get an user card details. 
  
  Create an API call to [Account User's Summary](https://streamamg.stoplight.io/docs/cloudpay/b3A6MTc0MTc5NTc-retrieve-the-user-s-summary) endpoint.

- validate that user's card details are updated.
  
  Create an API call to [Account User's Summary](https://streamamg.stoplight.io/docs/cloudpay/b3A6MTc0MTc5NTc-retrieve-the-user-s-summary) endpoint.
