<p align="right">

<img align="right" src="https://user-images.githubusercontent.com/57064943/163706360-f1d8e14a-aabd-40f2-90a0-0cdc0badf70c.png" height="20%" width="20%"/>

</p>

# `FoA Orion Keys API`


<br/><br/>
  

                               The Ford FG Falcon (FoA Orion) Secret Keys API


<sup>
Public API for the generation of response keys for 0x27 Security Access on Ford FG Falcon controllers.
<br/></sup>

![image](https://user-images.githubusercontent.com/57064943/163714778-8598c24a-6ae2-49f6-ba4c-42de94dfa025.png)


<img align="right" src="https://raw.githubusercontent.com/jakka351/FG-Falcon/master/resources/images/super-roo.png" height="20%" width="20%"/>

## Public Seed/Key Derivation API for Ford FG Falcon ECUs

OrionKeys is a public, security-conscious API that converts an **ECU Security Access seed** into the **correct response key** for Ford FG Falcon control modules. This enables diagnostics, service workflows, and healthy aftermarket tooling—so these vehicles can stay useful, safe, and enjoyable for years to come.

> **Mission:** Extend the life of FG Falcon vehicles by enabling compatible, diagnostic software and tools. By publishing a clear, secure, and stable API for seed→response derivation, we encourage maintenance and innovation, and help owners and repairers keep cars on the road rather than in the scrap yard.

---

## Table of Contents

- [What It Does](#what-it-does)  
- [Security Model](#security-model)  
- [Supported ECUs](#supported-ecus)  
- [Base URL](#base-url)  
- [Quick Start](#quick-start)  
  - [GET `/api/derive`](#get-apiderive)  
  - [POST `/api/derive`](#post-apiderive)  
  - [GET `/api/keys`](#get-apikeys)  
- [Request & Response Schemas](#request--response-schemas)  
- [Errors](#errors)  
- [Usage Notes](#usage-notes)  
- [Versioning & Stability](#versioning--stability)  
- [Ethics, Safety & Legal](#ethics-safety--legal)  
- [Contributing](#contributing)  
- [License](#license)

---

## What It Does

- Accepts a **3-byte ECU seed** (e.g., from a CGDS2003 `0x67 0x01` positive response).  
- Uses **ECU-specific secret keys** stored **server-side** to compute the **response key**.  
- Returns **only** the derived response (both `uint32` and hex string).  

Designed to integrate with J2534 tools, custom diagnostics, and service/engineering workflows.

---

## Security Model

- Secret keys live in a server-side `KeyStore` and **never** leave the server.  
- API responses expose **only** the derived result:
  ```json
  { "responseUInt32": 305419896, "responseHex": "0x12345678" }
  ```
- Inputs are validated (seed length and byte range).  

---

## Supported ECUs

The API targets the FG Falcon platform modules, including:

- Front Display Interface Module (FDIM)  
- Audio Control Module (ACM)  
- Audio Interface Module (AIM)  
- Bluetooth Phone Module (BPM)  
- Instrument Cluster (IC)  
- Body Electronics Module (BEM)  
- Transmission Control Module (TCM)  
- Powertrain Control Module (PCM)  
- Restraints Control Module (RCM)

---

## Base URL

```
https://orionkeys-fgbwb0habgdrhyh4.canadacentral-01.azurewebsites.net/API/
```

---

## Quick Start

### GET `/api/derive`

Derive a response key from a 3-byte seed and a known key identifier.

**Query Parameters**

- `seed` — Comma-separated 3 bytes. Each byte may be decimal (`18`) or hex (`0x12`, `AA`).  
  Examples: `seed=0x12,0x34,0x56` or `seed=18,52,86` or `seed=AA,BB,CC`.  
- `key` — A known key name (e.g., `IPC01`). Discover via `/api/keys`.

**Curl (PowerShell/Bash):**
```bash
curl -k 'https://orionkeys-fgbwb0habgdrhyh4.canadacentral-01.azurewebsites.net/API/derive?seed=AA,BB,CC&key=IPC01'
```

**Curl (Windows CMD.exe — note escaping of `&`):**
```bat
curl -k "https://orionkeys-fgbwb0habgdrhyh4.canadacentral-01.azurewebsites.net/API/derive?seed=AA,BB,CC^&key=IPC01"
```

**Portable form using `--get`:**
```bash
curl -k --get https://orionkeys-fgbwb0habgdrhyh4.canadacentral-01.azurewebsites.net/API/derive   --data-urlencode "seed=AA,BB,CC"   --data-urlencode "key=IPC01"
```

**Response 200**
```json
{
  "responseUInt32": 305419896,
  "responseHex": "0x12345678"
}
```

---

### POST `/api/derive`

Derive a response key using a JSON body.

**Request**
```json
{
  "seed": [170, 187, 204],
  "key": "IPC01"
}
```

- `seed` — array of exactly 3 integers, each `0..255`  

**Curl**
```bash
curl -k -X POST 'https://orionkeys-fgbwb0habgdrhyh4.canadacentral-01.azurewebsites.net/API/derive'   -H 'Content-Type: application/json'   -d '{"seed":[170,187,204],"key":"IPC01"}'
```

**Response 200**
```json
{
  "responseUInt32": 305419896,
  "responseHex": "0x12345678"
}
```

---

## Request & Response Schemas

### `GET /api/derive` (query)

- **Query**
  - `seed`: `string` (e.g., `"AA,BB,CC"` or `"0x12,0x34,0x56"` or `"18,52,86"`)
  - `key`: `string` (e.g., `"IPC01"`)

- **200 OK**
  ```json
  {
    "responseUInt32": 0,
    "responseHex": "0x00000000"
  }
  ```

- **400 Bad Request**
  ```json
  { "error": "Seed must have exactly 3 bytes, e.g. '0x12,0x34,0x56'" }
  ```

### `POST /api/derive` (JSON)

- **Body**
  ```json
  {
    "seed": [0, 0, 0],     // exactly 3 integers, each 0..255
    "key": "IPC01"
  }
  ```

- **200 OK**
  ```json
  {
    "responseUInt32": 0,
    "responseHex": "0x00000000"
  }
  ```

- **400 Bad Request**
  ```json
  { "error": "Key name is required." }
  ```

### `GET /api/keys`

- **200 OK**
  ```json
  ["IPC01", "PCM01", "..."]
  ```

---

## Errors

The API returns structured errors with `400 Bad Request` when input fails validation.

Common messages:

- `Seed must have exactly 3 bytes, e.g. '0x12,0x34,0x56'`  
- `Seed bytes must be 0..255.`  
- `Invalid seed format: <details>`  
- `Key name is required.`  
- `Unknown key '<name>'. Use /api/keys for options.`


## Versioning & Stability

- Minimal endpoints intended to remain stable:
  - `GET /api/derive`
  - `POST /api/derive`
- Backward-compatible enhancements may add headers, metadata, or auth without breaking existing clients.
- Track changes via release tags and changelog.

---

## Contributing

Issues and PRs that improve documentation, validation, observability, or deployment hardening are welcome.

---

