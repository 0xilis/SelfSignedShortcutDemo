# Self-Signed Shortcut Demo

This is a demo of a shortcut signed using my own private key. At first I thought this was a vulnerability, but after a couple of months Apple closed the report.

This requires someone to import a certificate that the shortcut is signed under, it is not a security issue. That's not saying it won't be changed in the future ever, but it's not needed for the current behavior to change and likely will only happen when Shortcuts team has nothing better to do (and they seem quite busy).

The shortcut that I signed is not mine but a random one I found on my hard drive. The certificate authority is mine. These files date back to March 5, 2024 16:26 EST (possibly earlier).

To my knowledge, at the time of writing I am the only one to do this. No one else looked into shortcut signing and what WorkflowKit does with it (at least not publicly).
