# 🔐 Xodus — Secure Cross-Platform File Sharing & Media Streaming

> **Role:** Hardware–software architect, security engineer, and UI developer  
> **Focus:** Local-first, privacy-preserving, encrypted file transfer

---

## 1. Project Overview

**Xodus** is a secure, cross-platform file-sharing and media-streaming system built around a custom USB dongle and companion apps for **Windows, macOS, Android, and iOS**.

The goal of Xodus is to provide **fast, private file transfers and media streaming without relying on cloud infrastructure**. All communication is local or optionally tunneled through a hardened WireGuard configuration, giving users and organizations strong control over their data.

---

## 2. Problem & Security Motivation

Traditional file-sharing tools often:
- Depend heavily on cloud storage and third-party servers
- Increase the attack surface (exposed APIs, misconfigurations, external data residency)
- Introduce recurring subscription costs and ongoing privacy risks

**Xodus** was designed to solve these issues by:

1. **Keeping data local** — transfers happen over local networks or through a controlled VPN tunnel, not via public cloud storage.  
2. **Minimizing external dependencies** — everything needed runs from the Xodus USB dongle or the apps, reducing supply chain and infrastructure risk.  
3. **Making secure workflows simple** — QR-based pairing, auto-launch scripts, and a clear UI ensure non-technical users can still benefit from strong security.

---

## 3. Security Architecture

### 3.1 Offline-First Design

- By default, Xodus uses **local networking only** (same-network communication between devices).  
- No third-party cloud storage: files are not uploaded to external servers.  
- The system can be used fully offline inside a LAN, which is ideal for organizations with strict data control requirements.

This design helps protect the **CIA triad**:

- **Confidentiality** – data never leaves controlled networks by default.  
- **Integrity** – transfers occur directly between trusted endpoints.  
- **Availability** – no dependency on external services or internet connectivity.

---

### 3.2 WireGuard-Based Secure Mode

For environments that require secure remote access or encrypted tunnels, the Xodus USB includes a **portable WireGuard implementation**. This can be launched directly from the dongle, without needing installation or admin-heavy setup.

#### Directory Structure (on the Xodus USB)

```text
Xodus/
  wireguard/
    wireguard-portable.exe
    WinTun.dll
    xodus-wg.conf
  xodus/
    XodusApp.exe
  run_xodus_secure.cmd
```

#### WireGuard Client Configuration — `XodusUSB/wireguard/xodus-wg.conf`

```ini
[Interface]
Address = 10.50.0.2/32
PrivateKey = <CLIENT_PRIVATE_KEY>
DNS = 1.1.1.1

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = XODUS_SERVER_PUBLIC_IP:51820
AllowedIPs = XODUS_SERVER_PUBLIC_IP/32
PersistentKeepalive = 25
```

- **Interface**  
  - `Address` assigns a static VPN IP for the client.  
  - `PrivateKey` is stored securely on the USB; not exposed in logs.  
  - `DNS` routed through a trusted resolver (e.g., 1.1.1.1).

- **Peer**  
  - `PublicKey` is the server’s WireGuard public key for mutual authentication.  
  - `Endpoint` defines the public IP and port of the WireGuard server.  
  - `AllowedIPs` restricts traffic to specific addresses (principle of least privilege).  
  - `PersistentKeepalive` maintains tunnel stability across NAT.

#### Secure Auto-Launch Script — `run_xodus_secure.cmd`

```cmd
@echo off
setlocal

REM Path to this USB stick
set ROOT=%~dp0

REM Paths to WG portable
set WG_EXE=%ROOT%wireguard\wireguard-portable.exe
set WG_CONF=%ROOT%wireguard\xodus-wg.conf

echo Starting Portable WireGuard...
"%WG_EXE%" /quick "%WG_CONF%"

echo Starting Xodus...
start "" "%ROOT%xodus\XodusApp.exe"

echo.
echo When you close Xodus, press any key to stop WireGuard.
pause >nul

echo Stopping Portable WireGuard...
"%WG_EXE%" /stop "%WG_CONF%"

echo Done.
endlocal
```

**Security & usability benefits:**  
- Entire secure workflow is **self-contained** on the USB device.  
- No need to install WireGuard system-wide.  
- Users simply run one script: it securely brings up the VPN tunnel and starts Xodus, then tears down the tunnel when finished.  
- Reduces configuration errors that commonly lead to misconfigured VPNs.

---

## 4. Application Features (Security-Focused)

- 🌐 **Cross-platform support** – Android, iOS, macOS, and Windows.  
- 🔒 **Secure local file sharing** – Devices discover each other securely on the same network.  
- 🎧 **Encrypted media streaming** – Stream audio/video from computer to mobile without saving files permanently on the phone.  
- 💾 **Custom USB hardware integration** – Dongle acts as portable secure endpoint/server.  
- 🧠 **Offline-first design** – Core functionality works with no internet.  
- 🔐 **Optional WireGuard tunnel** – For remote or untrusted-network use.

---

## 5. Cybersecurity Relevance

Xodus demonstrates practical security engineering skills across multiple layers:

1. **Network Security**
   - Local-only communication patterns.  
   - Encrypted tunnels via WireGuard.  
   - Controlled device discovery and pairing.

2. **System & Platform Security**
   - Portable, non-installing VPN and app packaging.  
   - Principle of least privilege with `AllowedIPs` and app-level permissions.  
   - Isolation of keys and configuration files on the USB device.

3. **Secure UX**
   - QR-based pairing reduces chances of user error when connecting devices.  
   - Single-script secure startup flow.  
   - Clear separation between “standard local mode” and “secure VPN mode”.

4. **Privacy by Design**
   - No cloud storage, no third-party data processing.  
   - Full control over where data physically resides.  
   - Architecture well-suited for organizations with data residency or compliance requirements.

---

## 6. Visual Case Study

> **Note:** Image paths here are placeholders; they can be replaced with actual hosted URLs or asset paths in your portfolio.

### 6.1 Hardware Close-Up

Showcasing the custom Xodus USB dongle used as a secure, portable endpoint.

```md
![Xodus USB dongle close-up](https://scontent-mia5-1.xx.fbcdn.net/v/t39.30808-6/469042872_549053107971420_2331577392041970834_n.jpg?_nc_cat=101&ccb=1-7&_nc_sid=127cfc&_nc_ohc=Z_P7AMp2ND0Q7kNvwEj26Ev&_nc_oc=AdlOTnD3rgR7Xv2s1JKjccw6kpgbDImQQfOlCaD6KlVF5ltyjzPdehsV2Rw1sHnKfR8&_nc_zt=23&_nc_ht=scontent-mia5-1.xx&_nc_gid=U9636KPVhcW2Q_a_T1t0Bg&oh=00_Afj0p_R2hAvGTePdbSC9K5-oUDlFJwSc8h-IsN0ThozJew&oe=69253252)
```

### 6.2 Multi-Device Secure Workflow

Laptop running the Xodus desktop app with the dongle attached, alongside a mobile device showing the Xodus mobile app.

```md
![Xodus app running on laptop and phone with Xodus USB dongle attached](https://scontent-mia3-2.xx.fbcdn.net/v/t39.30808-6/469204704_549052581304806_7071034171603607273_n.jpg?_nc_cat=105&ccb=1-7&_nc_sid=127cfc&_nc_ohc=vVzJ4Nv5clgQ7kNvwHo7IA1&_nc_oc=AdnYGjmvFcGYYlWB8MhmO2qoG9fwB5C2AZPIhyIevZf02bOfcqXghttP0Tt6PDR0sFE&_nc_zt=23&_nc_ht=scontent-mia3-2.xx&_nc_gid=mXqKzI1Ivq930XXBdwTjuw&oh=00_AfgOxJ1TQRt45wXMVtdJIivTtb6YmMsBP8UMThS27zHMWA&oe=69250821)
```

### 6.3 System Overview & Manual

A page from the user manual explaining how to plug in the device, open the app, scan the QR code, and start sending files securely.

```md
![Xodus quick start and feature overview manual](https://scontent-mia5-1.xx.fbcdn.net/v/t39.30808-6/475349918_588396270707857_5171924715690771691_n.jpg?_nc_cat=104&ccb=1-7&_nc_sid=127cfc&_nc_ohc=pcgaMN84FwcQ7kNvwFKUpuz&_nc_oc=AdmQwKqeHVbaweEj8nzdsJG7UTbVBYKxMGSwlpypiSnto04JLhnSNop7ex5L6WXEqh8&_nc_zt=23&_nc_ht=scontent-mia5-1.xx&_nc_gid=ZntygLnyVf_dS0OVIqu0VQ&oh=00_AfggBqGsggXQivTFu3wbter1HQ9yPiZyZADDlyinvpmfCg&oe=69252262)
```

### 6.4 Cross-Platform Demo

Tablet and laptop both connected via Xodus, demonstrating secure file transfer across platforms.

```md
![Cross-platform demo showing tablet and laptop using Xodus](https://scontent-mia3-1.xx.fbcdn.net/v/t39.30808-6/469069667_549052574638140_2809359909652229420_n.jpg?_nc_cat=111&ccb=1-7&_nc_sid=cc71e4&_nc_ohc=HkNkP-cEnbUQ7kNvwHmQxdV&_nc_oc=Adl9qAD1hA0foghQegn4P485v9j59ZTtVHT5UX_7eNdn6Iqd4ko4OVaFYKqtgqM1Pvc&_nc_zt=23&_nc_ht=scontent-mia3-1.xx&_nc_gid=_Xx9MiMuA8tyt0bkovW7Iw&oh=00_AfjB8sXEZWJbasIGYmfnjfb5KR66kEMw7TGxzfLx8n607w&oe=69251418)
```

> Additional screenshots (e.g., file lists or streaming views) can be included after blurring or cropping copyrighted media titles.

---

## 7. Links

> Replace these placeholders with your real URLs when publishing.

- 📱 **Xodus App – iOS (App Store)**  
  `https://share.google/J1Il8jfYU2POatWQu`  

- 🤖 **Xodus App – Android (APK Download)**  
  `https://share.google/d8n16TQXtOM731ssM`  

---

## 8. Personal Security Philosophy

I design systems to be **local-first and privacy-centric**:

- Users and organizations should retain **full ownership** and control over their data.  
- Cloud services are useful, but they also introduce ongoing costs and new attack surfaces.  
- When possible, I prefer architectures where critical workflows can run **fully offline** with **strong encryption** and **minimal external trust**.

My upcoming project, **Evocore**, continues this philosophy: an AI-powered system designed to operate entirely offline for maximum privacy and security, with local model execution and local knowledge bases.

---

## 9. Technologies & Skills Demonstrated

- **Languages & Frameworks**
  - Flutter & Dart (cross-platform UI)  
  - Swift / Kotlin (mobile integrations)  
  - Windows & macOS desktop development  

- **Networking & Security**
  - WireGuard configuration and automation  
  - Local network discovery and secure communication  
  - VPN tunneling and traffic scoping (AllowedIPs)  
  - Secure key handling and configuration management  

- **Systems & Hardware**
  - USB communication and file I/O  
  - Portable, self-contained application design  
  - Hardware–software integration and deployment

---

## 10. About the Author

**Vanel Cuffie**  
Cybersecurity & Software Developer  

- 15+ years of professional experience as a computer graphic artist.  
- Transitioning into cybersecurity with a focus on secure systems, ethical hacking, and privacy-first product design.  
- Passionate about building tools that combine **strong security**, **good UX**, and **practical real-world deployment**.
