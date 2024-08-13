# Shortcuts/WorkflowKit: Signature Validation Bypass

# Needed to reproduce:

At least for the PoC I have provided, the user must add and accept a certificate in their Settings app. To my testing this works on any iOS and macOS version with Shortcut signing.

# Steps to test PoC / Exploit

1. Download Snoolie Certificate Authority.cer and install and accept it in settings.
2. Import self-signed.shortcut

# Steps to reproduce

1. Generate a ECDSA root certificate. Make sure you give it the extension OID 1.2.840.113635.100.18.1, it doesn't matter what value you give it as long as it exists.
2. Generate a Certificate Authority for said root certificate.
3. Extract the private key from your root certificate. This will be used to encrypt the shortcut.
4. Generate auth data, containing your root .cer data as the first element of the SigningCertificateChain array and the 2nd element will be the certificate authority.
5. Sign the shortcut with your private key and the auth data.
6. On the device, add your certificate authority to the keychain.
7. Import the shortcut.

# Expected results

The SigningCertificateChain will be detected that even though it's on the keychain, it wasn't issued by Apple and therefore returns invalid.
