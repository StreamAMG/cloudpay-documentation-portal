# FAQ

### How do I use the API to generate a token I can use to access media

The first action a client should take when trying to access entitled/restricted content is to create a session.

This can be achieve using one of the following methods.

- [Generate Session using Email & Password](../reference/CloudPay-API-Specification.yaml/paths/~1api~1v1~1session~1start/get)
- [Generate Session using Payload](../reference/CloudPay-API-Specification.yaml/paths/~1api~1v1~1session~1start/post)
- [Generate a SSO Session](../reference/CloudPay-API-Specification.yaml/paths/~1sso~1start~1%7Bredirect%7D/get)

Once you have successfully logged in and a session is created you should store this locally so you can pass your CloudPay token into the API for future requests.

```json 
{
  "AuthenticationToken": "YzVkNzAyZmItMTVmNy00OTliLTkwZDUtY2MzNjkyMmUyYWIyMjAyMDA2MjQwMDM1NTdDRTVCRDg2ODREODhBMjMyMTExOEUxRTI1M0UyQzg3RDhCMDVBNEUz15B098C6"
}
```

The next step is to create a kSession for the entry you would like to view, this will validate that you have a active session and are also entitled to access the entry you have requested.

- [Generate a kSession Token](../reference/CloudPay-API-Specification.yaml/paths/~1api~1v1~1session~1ksession/get)

```json
{

  "KSession": "OWE1MGI2NmQ5ZTc2MjA5NjllYzEzY2EzMGFmNzUzNmExOTVkYzdlYXwzMDAxMTkyOzMwMDExOTI7MTU5Mjk5MzQwMDswOzE1OTI5OTI4MDA7Vmlld2VyO3N2aWV3OjBfOHpiNnV3Nngsc2V0cm9sZTpQTEFZQkFDS19CQVNFX1JPTEUsd2lkZ2V0OjE7Ow=="
}
```

You should again store the ksession you have generated and use this to access the selected media item.
