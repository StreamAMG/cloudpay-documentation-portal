# Checkout Redirect

CloudPay provides clients with hosted webpages to make the checkout process simple.

The simplest mechanism to initiate the checkout process can be achieved by calling the "PurchaseUrl" supplied to you within the [Package List](../../reference/CloudPay-API-Specification.yaml/paths/~1api~1v1~1package/get) endpoint.

By default, upon completion of a purchase, the user will be redirected back to a static page which can be configured within your CloudPay instance. 

If you would like to redirect a user back to a dynamic web page this can be achieved by passing in the "success_url" query parameter to the [Checkout](../../reference/CloudPay-API-Specification.yaml/paths/~1sso~1startbasket~1%7BsubscriptionPlanId%7D/get) endpoint. 

### Sequence

The following diagram demonstrates a simple integration flow between a third-party application and the CloudPay API.
<img src="https://lucid.app/publicSegments/view/97c75dac-8548-432d-abff-45d9fedea938/image.png" alt="Sequence Diagram" width="900" style="align:center"/>
