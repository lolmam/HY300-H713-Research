# HY300 / H713 OTA Research Summary

This directory contains a reverse-engineering breakdown of the firmware update mechanisms used by the **HY300 Pro (Allwinner H713)** platform.

## Research Files

| File | Focus | Status |
| --- | --- | --- |
| **[HtcOtaUpdate_QZ_RESEARCH.md](https://github.com/lolmam/HY300-H713-Research/blob/main/OTA%20Updates/HtcOtaUpdate_QZ_RESEARCH.md)** | **Active System:** `triplesai.com` API analysis. | **Primary** |
| **[OTA_RESEARCH.md](https://github.com/lolmam/HY300-H713-Research/blob/main/OTA%20Updates/OTA_RESEARCH.md)** | **Legacy System:** `bigbigcloud.cn` (DeviceHive) analysis. | **Inactive/Timeout** |

---

## Key Findings

### 1. The Active Path (`triplesai.com`)

The current system relies on the `HtcOtaUpdate_QZ` app.

* **Authentication:** Uses a custom `Sign` header—a lowercase **SHA1 hash** of alphabetically sorted JSON body keys.
* **Endpoint:** `http://ota.triplesai.com:8080/V1/Ota/Check`.
* **Current State:** The server is reachable, but frequently returns `Code: 208` (Product does not exist), suggesting specific firmware strings are missing from their database.

### 2. The Legacy Path (`bigbigcloud.cn`)

A secondary system associated with `com.softwinner.update`.

* **Infrastructure:** Built on a **DeviceHive-based API** using both REST and WebSockets.
* **Current State:** Appears to be a legacy Allwinner component. During testing, these servers were unresponsive or timed out.
