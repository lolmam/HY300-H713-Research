# HtcOtaUpdate_QZ OTA Update Research

## 1. Overview
The `HtcOtaUpdate_QZ` application is the active component responsible for online update checks on the HY300 Pro. It communicates with a custom API hosted at `triplesai.com`.

## 2. API Details
- **Base URL:** `http://ota.triplesai.com:8080`
- **Endpoints:**
  - `POST /V1/Ota/Check`: Main version check.
  - `POST /V1/Ota/CheckDevice`: Device verification (likely for activation/whitelist).

## 3. Communication Protocol

### Headers
Every request includes:
- `AppID`: `1`
- `Timestamp`: Current Unix timestamp in milliseconds (e.g., `1772378114311`).
- `Sign`: SHA1 signature of the JSON body fields.
- `Content-Type`: `application/json; charset=utf-8`

### Signature Algorithm
The `Sign` header is generated as follows:
1. Take all fields from the JSON body.
2. Sort the keys alphabetically.
3. Concatenate key and value pairs into a single string (e.g., `Key1Value1Key2Value2`).
4. Skip fields with empty values or nulls.
5. Generate a SHA1 hash of the resulting string in lowercase.

**Example String for `Check`:**
`ChannelHY200Pro_en_MagcubicOS_public_EMMC_cyhIMEIHYTY62508092801MAC7C:28:64:4A:A8:3DModelaodinVersionprojector.20250721.172901`

### Registration/Check Body Fields
Used in `/V1/Ota/Check`:
- `Version`: Current OTA version string (e.g., `projector.20250721.172901`).
- `Channel`: `HY200Pro_en_MagcubicOS_public_EMMC_cyh`.
- `Model`: `aodin`.
- `IMEI`: Hardware Serial Number (e.g., `HYTY62508092801`).
- `MAC`: WiFi MAC address with colons (e.g., `7C:28:64:4A:A8:3D`).

Used in `/V1/Ota/CheckDevice`:
- `ExpNum`: Hardware Serial Number.

## 4. Response Codes
- `Code: 0`: Success / Device Verified.
- `Code: 208`: Product does not exist. 
    - *Note:* This error occurs on the physical projector as well, suggesting the specific combination of Model/Channel/Version is not registered on the server or the service for this model has been discontinued.

## 5. Summary Findings
The signature is verified to be a simple SHA1 of the body fields. The API at `ota.triplesai.com` is active, but the product database appears to lack entries for the current firmware string (`projector.20250721.172901`).
