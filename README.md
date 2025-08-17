# Sovereign Guard (MVP)
Hardware watchdog for secure server recovery using **ESP32-CAM** (firmware in **ESP-IDF/Arduino C/C++**) and a **Python** agent over USB serial.

## TL;DR
- **Device is the boss**: it enforces policy and escalation.
- **Agent reports health** and executes *friendly* actions when asked (restart service / reboot OS).
- If soft actions fail or time out, the **device hard power-cycles** via relay.

---

## Key facts
- **Firmware**: ESP-IDF.
- **USB**: ESP32-CAM’s micro‑USB is **USB‑UART only** (serial). It cannot be a USB HID keyboard.
  - Optional: **BLE HID** keyboard (Ctrl+Alt+Del) if the host has Bluetooth.
- **Key storage**: classic ESP32 has **no discrete secure element**.
  - MVP: store device keys in flash; keep serial physical access controlled.
  - Roadmap: enable **Secure Boot + Flash Encryption** (ESP-IDF) or add an external secure element (e.g., ATECC608A).

---

## Architecture
### Device (ESP32-CAM)
- UART @ 115200, JSON lines (cJSON/ArduinoJson)
- Relay control + timers, watchdog state machine (backoff, lockout)
- Optional SD audit log
- **Roadmap**: Secure Boot + Flash Encryption (IDF), BLE HID

### Server (Python)
- `pyserial` for USB, `psutil` for metrics, optional FastAPI UI
- Service whitelist for restarts (e.g., `myapp.service`, `nginx.service`)
- Optional signature verification for pre-signed command bundles

---

## Escalation ladder & protocol
**Levels:**
1. Device → Agent: `restart_service` (systemd restart)
2. Device → Agent: `reboot_os` (friendly reboot)
3. Device: hard **relay** power cycle

**Messages (one JSON per line):**
```
# Agent → Device heartbeat (every 10 s)
{"t":"hb","ts":1699999999,"cpu":0.42,"load1":1.8,"svc_ok":true}

# Device → Agent requests
{"t":"request","req_id":"b8e1","action":"restart_service","service":"myapp.service",
 "reason":"svc_unhealthy","deadline_ms":30000}

{"t":"request","req_id":"aa77","action":"reboot_os","reason":"degraded_persist","deadline_ms":90000}

# Agent → Device replies
{"t":"ack","req_id":"b8e1","status":"executing"}
{"t":"done","req_id":"b8e1","ok":true,"msg":"systemctl restart done"}
```

**Policy bits (device):** heartbeat timeout → request restart; on failure → request reboot; on failure/timeout → relay.
Use stability window (e.g., 90s), backoff (10 min), and lockout (e.g., 3 actions/hour).

---

## Getting started
1. **Flash firmware** (ESP-IDF/Arduino) with your GPIO config for the relay.
2. **Wire the relay** (respect PSU/motherboard timing; typical off 6–10 s).
3. **Start the agent**: `pip install pyserial psutil` then run your `main.py` loop sending heartbeat JSONs.
4. **Test**: stop the agent and observe device escalation.

---

## Roadmap
- ESP32‑S2/S3 or RP2040 for **USB composite** (CDC + HID)
- Secure Boot + Flash Encryption
- External secure element (ATECC608A) for at-rest key isolation
- Signed request/response (Ed25519/HMAC) with anti‑replay

---

## License
MIT
