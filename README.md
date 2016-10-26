## OAuthSAMLClient

Sample app that demonstrates the flow to get an OAuth Bearer token based on a SAML2.0 assertion for an SAP Cloud for customer tenant. The token obtained can be used for SSO while invoking OData services offered by SAP Cloud for customer tenant.


#### OAuth SAML bearer flow - Overview

A SAML2.0 assertion is an XML security token, generally issued by a SAML2.0 identity provider which is consumed by a SAML2.0 service provider who relies on its content to identify the assertion’s subject for security-related purposes.

OAuth SAML bearer flow is a mechanism by which a SAML2.0 assertion can be used to request an OAuth token from the SAP Cloud for customer tenant, which then be used to make authenticated calls to SAP C4C OData services.

Overall flow for the OAuth SAML bearer flow is as shown below:

![OAuth SAML bearer flow](https://raw.githubusercontent.com/venkyvb/OAuthSAMLClient/master/sections/oauth_saml_bearer_flow.png)

### Pre-requisites:

In your SAP Cloud for Customer tenant please register an:
* OAuth IDP (Identity provider) - Note, if you are generating your own SAML signing certificates as given below, using OpenSSL, please upload the __samlidp_selfsigned.cer__ as the primary signing certificate when registering the OAuth IDP in C4C.
* OAuth Client

The above mentioned activities can be performed in the Administration work-center in your SAP Cloud for customer tenant. 

In order for the sample to run the following things need to be done:
* Add a JKS key-store to the project (e.g. in the current example the key-store is called as venkyvb.jks).
* Adjust the settings in the **settings.properties** file as follows:
  * CLIENT_ID = valid client ID based on the OAuth client registration mentioned above
  * CLIENT_SECRET = valid client secret associated with the OAuth client registration
  * ISSUER = your app which is registered as an OAuth IDP in your SAP Cloud for Customer tenant
  * NAME_ID = named user in SAP Cloud for customer tenant for whom the OAuth token is being requested (note that in real scenarios this would be determined based on the current logged in user in your app)
  * ENTITY_ID = Tenant URL without the protocol (HTTPS://)
  * TOKEN_SERVICE_URL = https://your_tenant_url/sap/bc/sec/oauth2/token
  * KEY_STORE_PASS = JKS keystore password
  * ...

The token obtained in this step needs to be added as a part of the "Authorization" header for the OData calls. The header should looke like:
* Header name = "Authorization"
* Value = "Bearer " + access_token

Note that this is just an illustrative sample.
Happy coding !!


##### How to use OpenSSL to create the JKS key-store

In case if you are interested, this section outlines how you can use OpenSSL to generate the JKS key-store that would be used by the OAuth client to sign the generated SAML assertions. Note that this example creates a keystore with name __samlidp_keystore.jks__, with the key alias as __samlidp_cert__.

1) Generate signing key for the SAML IDP
```
openssl genrsa -aes256 -out samlidp.key 2048
```

2) Generate cert request for CA
```
openssl req -x509 -sha256 -new -key samlidp.key -out samlidp.csr
```

3) Self sign the certificate (validity of 10 years !!)
```
openssl x509 -sha256 -days 3652 -in samlidp.csr -signkey samlidp.key -out samlidp_selfsigned.cer
```

4) Create pkcs12 keystore
```
openssl pkcs12 -export -name samlidp_cert -in samlidp_selfsigned.cer -inkey samlidp.key -out samlidp_keystore.p12
```

5) Convert pkcs12 into JKS keystore
```
keytool -importkeystore -destkeystore samlidp_keystore.jks -srckeystore samlidp_keystore.p12 -srcstoretype pkcs12 -alias samlidp_cert
```

6) Verify if everything is correct (using the java keytool utility)
```
keytool -list -v -keystore samlidp_keystore.jks
```


## To use OAuth SAML Bearer flow from HCP

Define a new Destination in your SAP HANA Cloud Platform Account
```
Name=e.g. C4C
Type=HTTP
URL=C4C_URL
ProxyType=Internet
Cloud Connector Version=2
Authentication=OAuth2SAMLBearerAssertion
Audience=C4C_URL_WITHOUT_PROTOCOL
Client Key=<same as Token Service User>
Token Service URL=https://myNNNNNN.crm.ondemand.com/sap/bc/sec/oauth2/token
Token Service User=<oAuth client ID registered in C4C>
Token Service Password=<the password provided during OAuth client configuration, under field "Client Secret">
authnContextClassRef=urn:oasis:names:tc:SAML:2.0:ac:classes:PreviousSession
nameIdFormat=urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress
scope=UIWC:CC_HOME
```
Use the destination in your HCP app logic when you want to make API calls to C4C.
