# Desktop Client (Win/Mac/Linux)

The USBridge desktop application (**USBridge-Remote**) is the primary administrative interface for out-of-band KVM control, virtual media provisioning, and hardware-level snapshot management. It's open source (GPLv3): [github.com/USBridge-Technologies/USBridge-Remote](https://github.com/USBridge-Technologies/USBridge-Remote) — see its own [client documentation](https://github.com/USBridge-Technologies/USBridge-Remote/blob/main/client/docs/README.md) for build instructions and platform-specific notes.

The same client also connects to a [software-only Agent](https://github.com/USBridge-Technologies/USBridge-Remote/blob/main/agent/docs/README.md) instead of a hardware KVM, when you just need software remote desktop rather than BIOS-level access — see [Agent vs. Hardware KVM](https://github.com/USBridge-Technologies/USBridge-Remote/blob/main/agent/docs/README.md#agent-vs-hardware-kvm) for exactly what that trades off.

## Client Downloads

Native binaries are published on the [Releases page](https://github.com/USBridge-Technologies/USBridge-Remote/releases/latest):

* **Windows:** `.zip`
* **macOS:** `.dmg` (Apple Silicon)
* **Linux:** `.AppImage` (x86_64)

Prefer a zero-install option? See the [Web Client](./web-client.md) — no download required, runs in the browser with some feature/performance trade-offs.

---

## Connection Initialization

To initiate a remote session, input the appliance's assigned IP address into the client's connection manager (e.g., `192.168.1.110`), or scan the pairing QR code from the front panel — see [Initial Setup & Client Pairing](../1-getting-started/initial-setup.md).

Upon successful authentication, the interface provisions access to four tabs: **Control**, **Devices**, **Snapshots**, and **Scripts**.

### Account Login & Connections Sync (Optional)

The account icon at the top of the window lets you sign in with Google — this is a separate, optional USBridge account, unrelated to appliance pairing (see [Security & Authentication Model §6](../10-developer-api/security-model.md#6-usbridge-account-login--connections-sync--a-fifth-optional-fully-separate-system) for the full trust model). Once signed in you can:

* See which USBridge licenses (appliance or [Agent](https://github.com/USBridge-Technologies/USBridge-Remote/blob/main/agent/docs/README.md)) belong to your account.
* Set a **sync passphrase** to sync your saved-connections list (every appliance/agent you've added, including pairing secrets) across your own devices — end-to-end encrypted client-side ([§6.1](../10-developer-api/security-model.md#61-connections-sync-is-end-to-end-encrypted--the-backend-cannot-read-it)); we never hold the key that could decrypt it.

### 1. Devices Tab (Hardware Passthrough)

Governs the configuration of virtual peripherals and the mounting of remote storage media to the target host.

* **Input Emulation:** Configures composite USB HID injection for keyboard and pointer control, heavily optimized for deterministic behavior within BIOS and text-based UEFI environments.
* **Virtual Media Staging:** Facilitates the remote attachment of bootable `.iso` images or virtual drives over the network, entirely bypassing the need for physical media during bare-metal OS provisioning.

### 2. Control Tab (Live KVM)

The primary interactive workspace for remote administrative operations.

It renders the real-time UVC video stream captured from the target host and bi-directionally routes local keyboard and pointer inputs back to the server. This module is utilized extensively for direct BIOS interaction, pre-OS navigation, and bare-metal recovery sequences.

### 3. Snapshots Tab

Centralized telemetry for the appliance's storage — see [Creating & Managing Snapshots](../4-snapshots-state-management/creating-managing-snapshots.md) for the actual mount/recover workflow this tab drives.

| Storage Mechanic | Technical Implementation |
| :--- | :--- |
| **Data Persistence** | Block-level data retention is governed by the native Btrfs Copy-on-Write (CoW) algorithm, ensuring highly efficient delta storage. |
| **WORM Protection** | All snapshots surfaced in the client are structurally locked as immutable, read-only subvolumes. This guarantees absolute resilience against target-host ransomware encryption or accidental deletion. |
| **Cross-Platform Access** | The appliance exports the storage repository via standard Media Transfer Protocol (MTP) or as a generic block device. Administrators can parse snapshot files natively in Windows Explorer or macOS Finder without requiring third-party Btrfs drivers. |

### 4. Scripts Tab

Manage and run [Starlark automation scripts](../3-bios-in-terminal/scripting-automation.md) without leaving the client:

* **Create:** **New (SD)** / **New (eMMC)** buttons start a new script on whichever storage you pick.
* **Edit:** a built-in syntax-highlighted editor, with **Save** and **Run** right in the toolbar — test a change immediately without switching to the Control tab.
* **Delete:** removes a script (with a confirmation prompt first).
* **MCP Proxy:** this tab is also where you [start the local MCP proxy](../3-bios-in-terminal/mcp-ai-agents.md#option-a-the-client-apps-mcp-proxy) for AI agent access — a Start/Stop toggle plus a Copy button for the local endpoint URL.
