# Security Notice — Lindell 17

This implementation follows the **"security with abort"** model of the original
Lindell 17 protocol ([eprint 2017/552](https://eprint.iacr.org/2017/552.pdf)).
This is a deliberate design tradeoff of the protocol, not a defect of this
implementation. Under this model a corrupted party may abort after learning
information that the honest party does not. Integrators are therefore
responsible for the deployment safeguards below.

## Deployment Responsibility

When you integrate this protocol, you **must**:

1. **Monitor for aborts.** In an honest run the signing flow always verifies
   successfully. Therefore any single failure — `Step3` returning `false` — is
   conclusive evidence that the counterparty is behaving maliciously.

2. **Stop and migrate on the first abort.** On any such failure, immediately
   stop using the affected key pair and migrate the associated assets to a new
   wallet. Do not retry signing with the same key share.

Without this monitoring, a malicious counterparty can repeatedly query the
signing flow and use the abort responses as an oracle to recover the honest
party's private key share. Cutting the session off at the first abort breaks
this attack.
