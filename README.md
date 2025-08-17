# Sovereign Guard

Sovereign Guard is a hardware-based watchdog for secure server monitoring using an ESP32 and cryptographic authentication. It provides an air-gapped, secure way to monitor and manage servers, ensuring that your infrastructure is always under your control.

## Features

*   **Hardware Watchdog:** An ESP32-based device that monitors your server at the hardware level.
*   **Secure USB Communication:** All communication between the watchdog and the server is done over a USB connection, making it immune to network-based attacks.
*   **Cryptographic Authentication:** Uses Ed25519 signatures and PGP key management to ensure that only authorized users can control the watchdog.
*   **Multi-level Access Control:** A hierarchical security model with device, admin, and superuser levels of access.
*   **Hardware-Protected Key Storage:** Private keys are stored in the ESP32's secure element and can't be extracted.
*   **Open Source and Self-Hosted:** Sovereign Guard is fully open source and can be self-hosted, giving you complete control over your monitoring infrastructure.

## Architecture

The Sovereign Guard system consists of two main components: the Sovereign Guardian Module (the ESP32 device) and the Sovereign Server Stack (a Python daemon that runs on your server).

### Sovereign Guardian Module (ESP32)

*   `crypto_manager.py`: Manages Ed25519 key generation, message signing, and verification using the `mpy-mbedtls` library.
*   `usb_comm.py`: Handles communication with the server over a USB CDC-ACM connection.
*   `watchdog.py`: Controls the hardware relay for power cycling the server and monitors its health.
*   `metrics.py`: Stores OpenTelemetry metrics on an SD card using `usqlite`.

### Sovereign Server Stack (Python)

*   `device_manager.py`: Detects and authorizes the ESP32 watchdog using the `pyserial` library.
*   `metrics_exporter.py`: Monitors system health using `psutil` and OpenTelemetry.
*   `command_processor.py`: Verifies GPG signatures on commands sent to the watchdog.
*   `web_interface.py`: A FastAPI-based web interface for managing the watchdog and viewing metrics.

## Security

Sovereign Guard is designed with security as a top priority. Here are some of the key security features:

*   **Air-Gapped Communication:** The watchdog is connected to the server via USB, which means it's not exposed to the network and can't be attacked remotely.
*   **Hardware Security:** The ESP32's Secure Boot and Flash Encryption features are used to create a hardware root of trust, protecting the device from physical attacks.
*   **Multi-Level Authentication:** The system uses a multi-level authentication model that requires different levels of cryptographic verification for different actions.
*   **Key Management:** Private keys are generated and stored on the ESP32 and are never exposed. The system also supports key rotation and revocation.

## Getting Started

To get started with Sovereign Guard, you'll need an ESP32 device and a server to monitor. Here are the basic steps to get up and running:

1.  **Hardware Setup:** Flash the Sovereign Guard firmware onto your ESP32 device and connect it to your server via USB.
2.  **Software Installation:** Install the Python dependencies for the server daemon.
3.  **Configuration:** Configure the watchdog and the server daemon with your desired settings.
4.  **Deployment:** Start the server daemon and you're ready to go!

For more detailed instructions, please refer to the [Deployment Guide](index.html#deployment).

## Technologies Used

### ESP32

*   [MicroPython](https://micropython.org/)
*   [mpy-mbedtls](https://github.com/Carglglz/mpy-mbedtls)
*   [usqlite](https://github.com/spatialdude/usqlite)

### Server

*   [Python](https://www.python.org/)
*   [pyserial](https://pyserial.readthedocs.io/en/latest/)
*   [cryptography](https://cryptography.io/en/latest/)
*   [python-gnupg](https://pypi.org/project/python-gnupg/)
*   [FastAPI](https://fastapi.tiangolo.com/)
*   [OpenTelemetry](https://opentelemetry.io/)

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
