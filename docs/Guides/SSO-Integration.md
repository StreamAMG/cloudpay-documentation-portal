# SSO Integration

The CloudPay API supports Third-Party SSO using JWT.

Supported JWT Algorithms

 - Standard JWT (HMAC)
 - Public/Private key pair (RSA)
 - Amazon Cognito JWT

### Setup

In order to enable a JWT SSO integration we first must enable this on your account, for this we need a number of details related to your SSO provider.

The following details are required;

 - Choice of Algorithm
 - Sharing of Secrets
 - Claims Map (Unique Identifier, First Name, Last Name, Email Address)

### Sequence

The following diagram demonstrates a simple integration flow between a third-party application and the CloudPay API.
<img src="https://lucid.app/publicSegments/view/269ea5cf-c3f7-411a-a37b-0100c7c7005a/image.png" alt="Sequence Diagram" width="900" style="align:center"/>

### Guide

The first part of the integration relies on the client to authorise the user and generate the users JWT Token on their hosted applications and authentication servers.

Once a user has a valid token this should be passed to the [Generate a SSO Session Endpoint](../../reference/CloudPay-API-Specification.yaml/paths/~1sso~1start~1%7Bredirect%7D/get) to initialise the CloudPay API Session.

> Currently this implementation makes use of the Set-Cookies response header along with HTTP Redirection header to store the CloudPay Authorisation Token within the browsers Cookie store. 

Once a CloudPay Session has been established the Cookie or Session token can be used throughout the API to provide trust when requesting access to other API functionality. 

> **Note:** *The CloudPay Session has the same signature as the Third-Party JWT Token*