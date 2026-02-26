# 🖥️ macOS Custom Icon Fix for UniFi UNAS

A guide to transforming the generic "Blue Screen" PC icon in macOS Finder into a professional **RackMac** (Server) icon by modifying Samba and Avahi configurations.

---

## 🛠️ Prerequisites

* **SSH Access:** Enabled via the UniFi OS settings.
* **Terminal:** Access to a CLI (macOS Terminal, iTerm2, etc.).
* **Root/Sudo Privileges:** Required to modify system configuration files.

---

## 🚀 Implementation

### 1. Samba Configuration (`smb.conf`)
The `fruit` module is an Apple-specific extension for Samba that allows macOS to recognize specific device metadata and performance optimizations.

1.  **SSH into your UNAS:**
    ```bash
    ssh root@<your-unas-ip>
    ```
2.  **Open the Samba configuration file:**
    ```bash
    sudo vi /etc/samba/smb.conf
    ```
3.  **Update the `[global]` section:**
    Ensure the following parameters are present to signal the hardware type to macOS:
    Some of these settings exist already towards the bottom of global
    ```ini
    [global]
    vfs objects = fruit full_audit streams_xattr
    fruit:aapl = yes
    fruit:nfs_aces = no
    fruit:model = rackmount
    fruit:wipe_intentionally_left_blank_rfork = yes
    fruit:posix_rename = yes
    fruit:veto_appledouble = no

    ```
5.  **Restart the Samba service:**
    ```bash
    systemctl restart smbd
    ```

---

### 2. Avahi Service Broadcast (Bonjour)
macOS often prioritizes **mDNS (Bonjour)** for Finder icons. Adding a `device-info` record ensures the icon appears even before the drive is mounted.

1.  **Edit the SMB service file:**
    ```bash
    sudo vi /etc/avahi/services/samba.service
    ```
2.  **Insert the `<txt-record>` for the model:**
    ```xml
    <?xml version="1.0" standalone='no'?>
    <!DOCTYPE service-group SYSTEM "avahi-service.dtd">
    <service-group>
      <name replace-wildcards="yes">%h</name>
      <service>
        <type>_smb._tcp</type>
        <port>445</port>
      </service>
      <service>
        <type>_device-info._tcp</type>
        <port>0</port>
        <txt-record>model=RackMac</txt-record>
      </service>
    </service-group>
    ```
    leave the folders before the last ```</service-group>``` alone
    
4.  **Restart the Avahi daemon:**
    ```bash
    systemctl restart avahi-daemon
    ```
---

** Restart SAMBA in settings/services on  Unifi web interface **

## 🎨 Icon Reference Table
You can change the `model=` string to match your preferred hardware aesthetic:

| String | Finder Icon Description |
| :--- | :--- |
| **`RackMac`** | **Xserve / Silver Rack Server (Classic)** |
| `MacPro7,1` | 2019 "Cheese Grater" Tower |
| `Macmini` | Silver Unibody Mac Mini |
| `Xserve` | Original Apple Xserve Blade |
| `iMac` | Modern Slim-profile Desktop |

---

## ⚠️ Important Notes
* **Persistence:** UniFi firmware updates frequently overwrite `/etc/samba/smb.conf`. It is recommended to keep a backup of these changes.
* **Finder Refresh:** If the icon doesn't update immediately, force-quit Finder (`Option` + Right Click Finder icon -> **Relaunch**) or clear the macOS Icon Cache.

Disclaimer
This is a community project and is not affiliated with Unifi. Use at your own risk.
---
