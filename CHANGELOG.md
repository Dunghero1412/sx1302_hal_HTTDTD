# Changelog

---

## [v2.1.0]

### Overview

This release targets **USB Corecell only** — no changes for SPI connection types. The USB-SPI bridge firmware running on the STM32 MCU has been updated for API clean-up and robustness improvements.

### Changes

**MCU — USB-SPI Bridge Firmware `v1.0.0`**

- Removed obsolete commands: `ORDER_ID__REQ_SPI`, `ORDER_ID__ACK_SPI`
- Shifted command index (`ORDER_ID__REQ_MULTIPLE_SPI`, `ORDER_ID__ACK_MULTIPLE_SPI`) following obsolete command removal
- Command parser now sends `ORDER_ID__UNKNOW_CMD` on receipt of a malformed command size
- Implemented `Error_Handler()` to reset the MCU upon fatal error
- Fixed a potential roll-over issue in `read_write_spi()`
- Increased delay tolerance for host feedback on USB transfers
- General code clean-up: typos fixed, comments added

**HAL — Command Interface updated for MCU Firmware `v1.0.0`**

- Removed obsolete commands from `order_id_e` enum
- Shifted command enum index to align with USB-SPI bridge update
- Removed unused `decode_ack_spi_access()` function
- Added timing debug information under `DEBUG_MCU`

---

## v2.0.2

### Overview

Fixed AGC firmware version check for `sx1255`/`sx1257`-based platforms (full-duplex gateways).

### Changes

- **HAL:** AGC firmware version for `sx1255`/`sx1257`-based gateways set to `v6`
- **HAL:** Minor cosmetic changes and typo fixes

---

## v2.0.1

### Overview

The fine timestamping feature has been fully validated with this release.

### Changes

- **HAL:** Adjusted the `freq_offset` field of received packets to account for channel IF resolution error
- **HAL:** Refined fine timestamp offset relative to Gateway v2, incorporating the frequency offset of the received packet
- **HAL:** Fixed preamble length for FSK downlinks
- **MCU:** Removed binary compiled in debug mode
- **util_spectral_scan:** Fixed `nb_scan` input argument which was previously ignored

---

## v2.0.0

### New Features

- Added support for **USB interface** between the host and the concentrator (sx1250-based concentrators only)
- Added support for **Listen-Before-Talk (LBT)** for the AS923 region, using the additional sx1261 radio from the Semtech Corecell reference design v3
- Added support for **Spectral Scan** with the additional sx1261 radio from the Semtech Corecell reference design v3
- Added support for the **SX1303 chip**, enabling Fine Timestamping
- Merged `master-fdd-cn490` branch to include support for the CN490 Full-Duplex reference design (incorporates releases v1.1.0, v1.1.1, v1.1.2)

### Changes

**HAL**

- Reworked the complete communication layer; introduced new `loragw_com` module to handle USB/SPI interface switching with aligned function prototypes for sx125x, sx1250, and sx1261 radios. USB mode groups SPI write commands to the STM32 MCU to optimize latency during time-critical configuration phases
- Added preliminary support for **Fine Timestamping** for TDOA localization
- Updated AGC firmware to `v6`: added configurable PA start delay and Listen-Before-Talk support
- Added new API function `lgw_demod_setconf()` for global demodulator settings
- Added new API functions for Spectral Scan

**Packet Forwarder**

- Interface type is now configurable via `global_conf.json`: `com_type` accepts `"USB"` or `"SPI"`
- Updated fine timestamping configuration parameters in `global_conf.json`
- Added configuration sections for Spectral Scan and Listen-Before-Talk features
- Added a background spectral scan thread as a usage example, demonstrating non-interfering use of the Spectral Scan API alongside normal gateway operations
- Added `nhdr` field parsing from `txpk` JSON downlink requests to support beacon requests from a Network Server
- Added `chan_multiSF_All` in `global_conf.json` to select which spreading factors are enabled for multi-SF demodulators
- Updated `PROTOCOL.md` to v1.6

**Tools**

- Added `util_spectral_scan`: a standalone spectral scanner utility

### Validation Note

> This release has been validated on the Semtech Corecell reference design v3 with USB interface.

---

## v1.1.2

> Integrated into **v2.0.0** from the `master-fdd-cn490` branch.

- **Packet Forwarder:** Updated `global_conf.json.sx1255.CN490.full-duplex` with RSSI temperature compensation coefficients and corrected RSSI offset for radio 1

---

## v1.1.1

> Integrated into **v2.0.0** from the `master-fdd-cn490` branch.

- **HAL:** Updated SX1302 LNA/PA LUT configuration for the Full-Duplex CN490 reference design
- **test_loragw_hal_rx/tx:** Added `--fdd` option to enable Full-Duplex mode
- **Packet Forwarder:** Updated `global_conf.json.sx1255.CN490.full-duplex` for CN490 reference design

---

## v1.1.0

> Integrated into **v2.0.0** from the `master-fdd-cn490` branch.

- **HAL:** Added support for CN490 Full-Duplex reference design

---

## v1.0.5

- **HAL:** Fixed packet timestamp issue causing time jumps under specific conditions
- **HAL:** Added workaround for hardware issue when reading 32-bit registers (timestamp, RX buffer byte count, etc.)
- **HAL:** Fixed potential endless loop in `sx1302_tx_abort()` on SPI access failure
- **Packet Forwarder:** Added `global_conf.json.sx1250.US915` for the US915 band
- **test_hal_rx:** Added command-line option to specify an RSSI offset

---

## v1.0.4

- Added missing `LICENSE.TXT` file
- **HAL & Packet Forwarder:** Added support for sx1250-based reference design for the CN490 region
- **Packet Forwarder:** Disabled beaconing by default

---

## v1.0.3

- **HAL:** Fixed scheduled downlink time precision by accounting for TX start delay
- **HAL:** Fixed timestamp correction calculation for BW250 and BW500
- **HAL:** Fixed possible buffer overflow in `lgw_receive()`
- **HAL:** Packets retained in the RX buffer when the receive buffer is too small; remaining packets are fetched on subsequent `lgw_receive()` calls (aligned with SX1301 behaviour)

---

## v1.0.2

- Fixed compilation warnings reported by recent GCC versions
- Reworked temperature sensor handling
- Removed unused files
- Added instructions and configuration files for packet forwarder auto-start via `systemd`
- Added SX1250 radio calibration at startup

---

## v1.0.1

- **Packet Forwarder:** Updated TX gain LUT in `global_conf.json.sx1250` with proper calibration values

---

## v1.0.0

- **HAL:** Initial official release for the SX1302 CoreCell Reference Design

---

## v0.0.1

- **HAL:** Initial private release for the TAP program
