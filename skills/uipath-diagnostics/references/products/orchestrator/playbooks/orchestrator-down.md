---
product: orchestrator
scenario: Orchestrator completely inaccessible or returning 500 errors
level: product
---

# Orchestrator Down

## Symptoms

- Browser displays "Oops! Orchestrator Is Down"
- Robots and Studio cannot reach Orchestrator
- HTTP 500.30 or connection forcibly closed errors

## Triage

- Confirm the Orchestrator URL is unreachable (not a network/DNS issue on the client side)
- Check if other platform services (Portal, Identity) are also down

## Hypothesis Generation

### Forcibly Closed Connection
- IIS app pool recycled
- Host machine reboot
- Network connection lost between load balancer and Orchestrator nodes

### 500.30 ASP.NET Core Startup Failure
- Legacy ASP.NET config sections left in dll.config after upgrade to 2021.10+
- Error: "Unrecognized configuration section system.web"
- Fix: remove legacy `system.web` sections from dll.config

### Identity Server Redirect Loop
- Browser shows "Too Many Redirects" at login
- SSL certificate mismatch with IdentityServer.Integration.Authority
- ASP.NET Core Web Hosting Bundle missing
- Error Code: 0x8007000d

## Testing

- Check IIS application pool status and event logs
- Verify SSL certificate validity and trust chain
- Check if ASP.NET Core Web Hosting Bundle is installed
- For clustered deployments: verify NTP sync < 1 second across nodes

## Resolution

- Restart IIS application pool
- Fix SSL certificate issues
- Install ASP.NET Core Web Hosting Bundle if missing
- Remove legacy config sections after upgrade
