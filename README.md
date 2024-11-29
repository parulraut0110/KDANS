A certificate chain created with self-signed top-level RootCA,  an intermediate CA (IntermediateCA) signed by RootCA, and End-Entity Certificate (OAuth2Server.cer / OAuth2Client.cer)  signed by IntermediateCA.

**What you will find**

1) Use of java keytool to create keypairs, self signed root certificates, intermediate certificates, and end-entity-certificate with basic constraints and attributes and generating a certificate chain similar to a certificate authority and configuring the certificates for use with Java applications and web browsers.

2) Creating Keystores, importing and exporting certificates to keystores  

3) Placement of the generated root certificate in 'cacert' for java application

4) Placement of root certificate in the system directory for browser identification

5) Aliasing the localhost to a server domain and its adaption in the certificate attribute.


**By following these steps, one can successfully create a certificate chain with a self-signed Root CA, configure Java and system trust stores, and set up a local domain alias. This setup allows Java applications and web browsers to recognize and trust the self-signed certificates, enabling secure communications during development.**


[Medium ](https://medium.com/@parulraut0110/self-signed-digital-certificate-chain-using-java-keytool-756caa574c62)
