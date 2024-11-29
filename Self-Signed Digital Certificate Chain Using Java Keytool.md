
# Self-Signed Digital Certificate Chain Using Java Keytool

A certificate chain is created with a self-signed top-level Root CA, an Intermediate CA (IntermediateCA) signed by RootCA, and an End-Entity Certificate (OAuth2Server.cer / OAuth2Client.cer) signed by IntermediateCA. The following steps illustrate the creation of three certificates.

To create a Root CA keypair and store it in a keystore:
```bash
keytool -genkeypair -alias root -dname "cn=RootCA, ou=Root_CertificateAuthority, o=CertificateAuthority, c=IN" -keyalg RSA -keystore c:/caKeyStore.keystore -storepass password -keypass password -ext KeyUsage=digitalSignature,keyCertSign -ext BasicConstraints=ca:true,PathLen:3 -validity 1000
```
Ensure to include the `BasicConstraints=ca:true` attribute. Without it, the root certificate would not be trusted by the browser or Java as a trusted signing authority.

To create an Intermediate CA keypair:
```bash
keytool -genkeypair -alias intermediate -dname "cn=IntermediateCA, ou=Intermediate_CertificateAuthority, o=CertificateAuthority, c=IN" -keyalg RSA -keystore c:/caKeyStore.keystore -storepass password -keypass password -ext KeyUsage=digitalSignature,keyCertSign -ext BasicConstraints=ca:true,PathLen:3 -validity 1000
```
The `PathLen` attribute signifies the maximum number of certificates that can be obtained in the certificate chain signed by the issuing CA certificate.

To create a server keypair and its keystore:
```bash
keytool -genkeypair -alias server -dname "cn=OAuth2Server, ou=Java, o=Oracle, c=IN" -keyalg RSA -keystore c:/oauth2server.jks -storepass password -keypass password -validity 1000 -ext KeyUsage=digitalSignature,dataEncipherment,keyEncipherment,keyAgreement -ext ExtendedKeyUsage=serverAuth,clientAuth
```

To create a certificate signing request (CSR) and certificate for IntermediateCA:
```bash
keytool -certreq -alias intermediate -storepass password -keystore c:/caKeyStore.keystore -keyalg RSA | keytool -alias root -gencert -ext san=dns:intermediate -storepass password -keystore c:/caKeyStore.keystore -keyalg RSA -validity 1000 -ext KeyUsage=digitalSignature,dataEncipherment,keyEncipherment,keyAgreement,keyCertSign -ext ExtendedKeyUsage=serverAuth,clientAuth -ext BasicConstraints=ca:true,PathLen:3 -rfc | keytool -alias intermediate -importcert -storepass password -keyalg RSA -keystore c:/caKeyStore.keystore
```

The keyword `SAN` (Subject Alternate Name) is used to configure the domain name of the server certificate as would be visible in the browser (modern browsers look for `SAN` to validate rather than `CN`).

To export the root certificate and intermediate certificate, and import them to the server keystore:
```bash
keytool -exportcert -alias root -storepass password -keystore c:/caKeyStore.keystore -validity 1000 | keytool -importcert -alias root -keystore c:/oauth2server.jks -storepass password -noprompt -trustcacerts

keytool -exportcert -alias intermediate -storepass password -keystore c:/caKeyStore.keystore -validity 1000 | keytool -importcert -alias intermediate -keystore c:/oauth2server.jks -storepass password -noprompt -trustcacerts
```

The order of importing certificates in the keystore is important and should be followed in the order **Root CA -> Intermediate CA -> Server Certificate**.

To create a CSR for the server and import it into the server keystore:
```bash
keytool -certreq -alias server -storepass password -keystore c:/oauth2server.jks -keyalg RSA | keytool -alias intermediate -gencert -ext san=dns:OAuth2Server -storepass password -keystore c:/caKeyStore.keystore -keyalg RSA -ext KeyUsage=digitalSignature,dataEncipherment,keyEncipherment,keyAgreement -ext ExtendedKeyUsage=serverAuth,clientAuth -rfc | keytool -alias server -importcert -storepass password -keyalg RSA -keystore c:/oauth2server.jks -validity 1000
```

To delete the imported CA certificates from the server keystore:
```bash
keytool -delete -alias root -keystore c:/oauth2server.jks -storepass password
keytool -delete -alias intermediate -keystore c:/oauth2server.jks -storepass password
```

To create a trust keystore with Root and Intermediate certificates:
```bash
keytool -exportcert -alias root -storepass password -keystore c:/caKeyStore.keystore | keytool -importcert -alias root -keystore c:/trust.jks -storepass password -trustcacerts -noprompt

keytool -exportcert -alias intermediate -storepass password -keystore c:/caKeyStore.keystore | keytool -importcert -alias intermediate -keystore c:/trust.jks -storepass password -trustcacerts -noprompt
```

To extract certificates:
```bash
keytool -exportcert -alias root -keystore c:/trust.jks -file c:/rootCA.cer -storepass password
keytool -exportcert -alias intermediate -keystore c:/trust.jks -file c:/intermediateCA.cer -storepass password
keytool -exportcert -alias server -keystore c:/oauth2server.jks -file c:/oauth2server.cer -storepass password
```

To import RootCA into JVM’s truststore (`cacerts`):
```bash
keytool -import -trustcacerts -keystore "C:/Program Files/Java/jdk-17.0.4/lib/security/cacerts" -storepass changeit -alias rootCA -file "C:/rootCA.cer"
```

To verify entries in your keystore:
```bash
keytool -list -v -keystore your-keystore-location -storepass your-keystore-password -alias alias-name
keytool -list -v -keystore "C:/caKeyStore.keystore" -storepass password -alias rootCA
```

To verify your import in `cacerts`:
```bash
keytool -list -v -keystore "C:/Program Files/Java/jdk-20/lib/security/cacerts" -alias rootCA
```

To import the Root Certificate into your browser’s trusted certificate directory:

1. Open the **Run** dialog box (`Win + R`) and type `certmgr.msc`.
2. Click **Trusted Root Certificate Authorities** in the left pane.
3. Right-click the **Certificates** subfolder, then click **All Tasks** -> **Import**.
4. Follow the **Certificate Import Wizard** to browse and select your certificate file (e.g., `rootCA.cer`).

This completes the process of generating and importing a self-signed certificate chain.
