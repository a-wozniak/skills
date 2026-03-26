---
product: orchestrator
scenario: Robots cannot connect to Orchestrator due to TLS/SSL certificate validation failure
level: product
---

# Remote Certificate Invalid

## Symptoms

- Robots cannot connect to Orchestrator
- TLS/SSL handshake failure
- Error: "The remote certificate is invalid according to the validation procedure"

## Triage

- Check if the issue affects all robots or specific machines
- Check if the Orchestrator SSL certificate was recently changed or renewed

## Testing

- Verify the SSL certificate on the Orchestrator URL is valid (not expired, correct hostname)
- Check if the certificate chain is complete (intermediate certificates present)
- On the robot machine, check if the certificate's root CA is in the Trusted Root Certification Authorities store

## Resolution

- Import the certificate (or its root CA) into the Trusted Root Certification Authorities store on the robot machine
- If the certificate was recently renewed, the new root CA may need to be distributed to all robot machines
- For self-signed certificates: import the self-signed cert into Trusted Root on every robot machine
