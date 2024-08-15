# Self-Signed Shortcut Demo

This is a demo of a shortcut signed using my own private key. At first I thought this was a vulnerability, but after a couple of months Apple closed the report.

This requires someone to import a certificate that the shortcut is signed under, it is not a security issue. That's not saying it won't be changed in the future ever, but it's not needed for the current behavior to change and likely will only happen when Shortcuts team has nothing better to do (and they seem quite busy).

The shortcut that I signed is not mine but a random one I found on my hard drive. The certificate authority is mine. These files date back to March 5, 2024 16:26 EST (possibly earlier).

To my knowledge, at the time of writing I am the only one to do this. No one else looked into shortcut signing and what WorkflowKit does with it (at least not publicly).

## How it works

Contact signing and iCloud signing both verify their signing chains in a different way. iCloud Signing (the entrypoint for this) is done in the validateSigningCertificateChainWithICloudIdentifier method, and looks like this:

```objc
-(BOOL)validateSigningCertificateChainWithICloudIdentifier:(NSString **)iCloudId error:(NSError **)err {
    /* TODO: Implement errs (really just WFShortcutSigningContextSigningCertificateChainFailureError() ) */
    WFSecurityLog("Validating Shortcut Signing Certificate Chain");
    NSArray <WFShortcutSigningCertificate *>* signingCertificateChain = [self signingCertificateChain];
    /* if_map is from IntentsFoundation.framework */
    NSArray* certificates = [signingCertificateChain if_map:^(WFShortcutSigningCertificate *item){
      [item certificate]; //WFShortcutSigningCertificate
    }];
    SecPolicyRef policy = SecPolicyCreateRevocation(kSecRevocationUseAnyAvailableMethod);
    SecTrustRef trust = 0;
    OSStatus res = SecTrustCreateWithCertificates((__bridge CFArrayRef)certificates, policy, &trust);
    if (res == 0 || (res != 0 && !trust)) {
        SecCertificateRef root = (__bridge SecCertificateRef)(certificates[0]);
        if (iCloudId) {
            CFStringRef rootCertName = 0;
            SecCertificateCopyCommonName(root, &rootCertName);
            *iCloudId = (__bridge NSString *)rootCertName;
        }
        CFErrorRef evaluateErr = 0;
        bool isValid = SecTrustEvaluateWithError(trust, &evaluateErr);
        if (isValid) {
            if (SecCertificateCopyExtensionValue(root, @"1.2.840.113635.100.18.1", 0)) {
                WFSecurityInfo("Shortcut Signing Certificate Chain Validated Successfully");
                return YES;
            } else {
                WFSecurityErrorF("Unrecognized Shortcut Signing Certificate: %@",root);
            }
        } else {
            WFSecurityErrorF("Failed to Evaluate Shortcut Signing Certificate Chain: %@",certificates);
        }
    } else {
        WFSecurityErrorF("Validating Shortcut Signing Certificate Chain Failed: %@",certificates)
    }
    return NO;
}
```

It checks that the certs are validated with any existing in the keychain, and to check that it is issued by Apple, it makes sure that the root certificate has extension OID `1.2.840.113635.100.18.1`. This means you can create your own certificate chain with a root cert (with that root certificate having `1.2.840.113635.100.18.1` added) and certificate authority, and add the certificate authority onto the system keychain, to which it will then validate the shortcut.

[libshortcutsign](https://github.com/0xilis/libshortcutsign) allows you to sign a shortcut with custom authData and a custom key.
