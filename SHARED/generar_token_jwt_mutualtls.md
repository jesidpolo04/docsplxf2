# APIM with entraID - JWT 

## Prerequisites

- Azure AD Application: Ensure you have registered an application in Azure AD and have the Client ID, Client Secret, and Tenant ID.

- Certificates: You need client and server certificates for mTLS.

- Configure JWT Validation in APIM:
Add the `validate-jwt` policy to your API in APIM to ensure that incoming requests contain a valid JWT token. Here’s an example configuration:

```XML
<inbound>
  <base />
  <validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized">
    <openid-config url="https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration" />
    <audiences>
      <audience>api://{your-api-client-id}</audience>
    </audiences>
    <issuers>
      <issuer>https://sts.windows.net/{tenant-id}/</issuer>
    </issuers>
  </validate-jwt>
</inbound>
```

## Steps to Request a JWT Token with mTLS

- Register an Application in Azure AD:

    Register your application in Azure AD and configure it to use client certificates for authentication.

- Generate and Upload Certificates:
    
    Generate a client certificate and upload it to your Azure AD application under the Certificates & secrets section.

- Request an Access Token:

    Use the following curl command to request an access token from Azure AD with mTLS:

        curl -X POST \
        -H "Content-Type: application/x-www-form-urlencoded" \
        --cert client-cert.pem \
        --key client-key.pem \
        -d "grant_type=client_credentials&client_id=<YOUR_CLIENT_ID>&client_secret=<YOUR_CLIENT_SECRET>&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default" \
        https://login.microsoftonline.com/<YOUR_TENANT_ID>/oauth2/v2.0/token

        Example Breakdown:
        grant_type: Specifies the OAuth 2.0 grant type. For client credentials flow, use client_credentials.
        client_id: The Application (client) ID of your registered Azure AD application.
        client_secret: The client secret generated for your application.
        scope: The scope of the access request. Use https://yourapiuri/.default.
        URL: The token endpoint URL for your Azure AD tenant.
        –cert: Path to the client certificate file.
        –key: Path to the client certificate private key file.

Token request URL example: https://login.microsoftonline.com/<YOUR_TENANT_ID>/oauth2/v2.0/token

Example Response:
If the request is successful, you will receive a JSON response containing the access token:

```json

{
  "token_type": "Bearer",
  "expires_in": 3600,
  "ext_expires_in": 3600,
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6..."
}
```
You can then use this access token to authenticate API requests by including it in the Authorization header:

    curl -H "Authorization: Bearer <YOUR_ACCESS_TOKEN>" https://yourapi.com/v1.0/resource


