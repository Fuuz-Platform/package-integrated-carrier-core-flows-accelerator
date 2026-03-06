# package-integrated-carrier-core-flows-accelerator

**Version:** 0.0.1
**Spec Version:** 2.0.0

---

## Overview

This package installs the core carrier-agnostic data flows for the Integrated Carrier shipping system. It provides the central routing logic that dispatches shipment requests to the correct carrier-specific handler, and the label print flow that interfaces with the Fuuz Device (printer) subsystem.

These flows are designed to work with any carrier registered in the `IntegratedCarrier` schema. Adding a new carrier only requires creating the appropriate `IntegratedCarrierRequestFlow` seed records and implementing carrier-specific API flows — the router itself does not change.

---

## Package Contents

```
integrated-carrier-core-flows/
├── manifest.json
├── package-data.json
├── install/                     6 install steps
├── preinstall/                  2 preinstall verification steps
└── postinstall/                 2 postinstall configuration steps
```

---

## Installed Flows (2 core flows)

### Integrated Carrier Router (`Integration`)

The central dispatch flow for all shipment operations. Receives a shipment request with carrier ID and request type, looks up the appropriate `IntegratedCarrierRequestFlow` record, and executes the matched carrier-specific flow (rate quote, label generation, void, tracking, etc.).

Key behavior:
- Queries `IntegratedCarrierRequestFlow` using `carrierId` + `requestTypeId` to find the handler flow ID
- Calls `$executeFlow(handlerFlowId, payload)` — enabling a plug-in model where new carriers register their flows without modifying the router
- Returns the carrier API response to the calling screen or flow
- Logs the request and response to `IntegratedCarrierRequest` for audit and reprint

### Integrated Carrier Print Labels Standard (`Integration`)

Handles standard label print operations after successful label generation:

- Receives the carrier label response (Base64 ZPL or PDF)
- Resolves the assigned printer device from the workcenter or shipment context
- Dispatches the label to the Fuuz Device print queue
- Supports thermal label printers (ZPL) and standard laser label formats (PDF)

---

## Install Process

**Preinstall (2 steps):** Verifies that neither `Integrated Carrier Router` nor `Integrated Carrier Print Labels Standard` flows already exist by ID and name, preventing duplicate installation.

**Install (6 steps):**
1. Creates the `Integrated Carrier Router` DataFlow header (inactive)
2. Creates and deploys the Router flow version
3. Creates the `Integrated Carrier Print Labels Standard` DataFlow header (inactive)
4. Creates and deploys the Print Labels flow version
5. Activates the Router flow
6. Activates the Print Labels flow

**Postinstall (2 steps):**
1. Registers the Router flow ID into the `IntegratedCarrierConnectionConfiguration` records for each carrier account
2. Validates that the flows are active and deployable

---

## Installation

1. Install `package-integrated-carrier-core-schema-accelerator` first
2. Import this package via Fuuz Package Manager
3. Verify both flows are active in the Fuuz Data Flow manager
4. Install carrier-specific packages (FedEx, UPS) to register handler flows with the router
5. Assign printers to workcenters via the Fuuz Device settings for label printing

---

## Dependencies

- **`package-integrated-carrier-core-schema-accelerator`** — required; must be installed first
- **Fuuz Device module** — required for label printing
- Carrier-specific packages must be installed AFTER this package

---

## Part of the Integrated Carrier Suite

| Package | Description |
|---------|-------------|
| `integrated-carrier-core-schema` | Data models |
| **integrated-carrier-core-flows** (this) | Router and print flows |
| `integrated-carrier-core-screens` | Shipment management screens |
| `integrated-carrier-fedex` | FedEx seed data and label flows |
| `integrated-carrier-ups` | UPS seed data and label flows |
| `integrated-carrier-addon-plex` | Plex ERP integration extension |

---

*Built on the [Fuuz Industrial Operations Platform](https://fuuz.com)*
