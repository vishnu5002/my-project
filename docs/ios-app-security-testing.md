# iOS Mobile Application Security Testing Guide

This guide covers setting up an environment to test iOS applications for authorized bug bounty programs and security assessments according to OWASP Mobile Application Security (MAS) standards.

---

## 1. Network Traffic Interception Setup

To inspect API calls and HTTP/HTTPS traffic between the target iOS app and backend servers:

### Step 1: Configure Intercepting Proxy
1. Launch your proxy (Burp Suite, Caido, or mitmproxy) on your testing computer.
2. In proxy settings, configure the listener to bind to **All interfaces** (or your computer's local Wi-Fi IP) on port `8080`.
3. Note your computer's local IP address (e.g., `192.168.1.50`).

### Step 2: Configure iPhone Wi-Fi Proxy
1. Ensure your iPhone and computer are connected to the same Wi-Fi network.
2. On your iPhone, open **Settings > Wi-Fi** and tap the **(i)** icon next to your network.
3. Scroll down to **Configure Proxy** and select **Manual**.
4. Enter:
   - **Server:** Your computer's local IP (e.g., `192.168.1.50`)
   - **Port:** `8080`
5. Tap **Save**.

### Step 3: Install & Trust Root CA Certificate
1. On your iPhone, open Safari and navigate to your proxy's CA download page (e.g., `http://burp/cert` or `http://<proxy-ip>:8080`).
2. Download the configuration profile when prompted.
3. Open **Settings > Profile Downloaded** and tap **Install**.
4. **Crucial Step:** Go to **Settings > General > About > Certificate Trust Settings**.
5. Under "Enable full trust for root certificates", locate your proxy certificate and toggle the switch to **On**.

---

## 2. Handling SSL Pinning & Anti-Tampering

Many production iOS applications implement SSL pinning to verify the server's certificate against hardcoded hashes. When pinning is enabled, the app will refuse connections through your intercepting proxy.

### Approach A: Non-Jailbroken Devices (IPA Repackaging)
If your iPhone is not jailbroken, you can patch the IPA file before installing it:
1. Extract the decrypted IPA file.
2. Inject `FridaGadget.dylib` into the application binary using tools like `insert_dylib` or `optool`.
3. Re-sign the modified app with your Apple Developer / free personal provisioning profile using `codesign` or `Sideloadly`.
4. Run the app on your device; FridaGadget will open a listening port over USB to allow dynamic hooking via [objection](https://github.com/sensepost/objection) (`objection explore` -> `ios sslpinning disable`).

### Approach B: Jailbroken Testing Device (Recommended for Research)
A dedicated test device with a jailbreak (e.g., Palera1n for checkm8 devices, or Dopamine for iOS 15.0–16.6.1) enables dynamic analysis:
- **Frida (`frida-server`)**: Install via Sileo/Zebra to dynamically hook runtime methods.
- **SSL Kill Switch 2**: Hooks low-level SecureTransport and Network framework functions to bypass pinning system-wide.
- **Filza File Manager**: Allows direct inspection of app sandboxes, `Info.plist` files, SQLite databases, and cached tokens.

---

## 3. Key iOS Bug Bounty Checkpoints (OWASP MASVS)

When auditing iOS apps, focus on the following primary areas:

1. **API Security & Logic Flaws:**
   - Review all API endpoints captured in your proxy history.
   - Test for Broken Object Level Authorization (BOLA/IDOR), missing rate limits, and unauthenticated endpoints.

2. **Insecure Data Storage:**
   - Inspect data stored in `/var/mobile/Containers/Data/Application/<UUID>/`.
   - Check if sensitive tokens, passwords, or PII are written to `Library/Preferences/*.plist`, local SQLite/Realm databases, or log files (`NSLog`).
   - Ensure sensitive credentials use the iOS Keychain with secure access control attributes (`kSecAttrAccessibleAfterFirstUnlock`).

3. **Custom URL Schemes & Deep Links:**
   - Inspect `CFBundleURLTypes` in `Info.plist`.
   - Test custom scheme handlers (e.g., `myapp://action?param=value`) for input validation issues, parameter tampering, or account takeover flows.

4. **Universal Links & Associated Domains:**
   - Check `applinks` definitions and verify that incoming deep link payloads are validated before triggering actions.
