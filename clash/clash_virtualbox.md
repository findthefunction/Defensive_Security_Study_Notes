**How to Route VirtualBox Traffic Through Host Machine's Clash Instance**

This document explains how to configure a VirtualBox virtual machine (VM) to route its internet traffic through the Clash proxy running on the host machine. This allows the VM to leverage the host's proxy settings and routing capabilities.

Some Clash links for posterity
https://github.com/clash-hub/clash_for_windows_pkg/releases (works through the wall)
https://archive.org/download/clash_for_windows_pkg

---

### **Prerequisites**
1. Clash is installed and running on the host machine.
2. Clash is configured with `allow-lan: true` in its configuration file.
3. You have a VirtualBox VM (e.g., Kali Linux) set up and running.
4. Basic knowledge of VirtualBox and networking.

---

### **Steps to Configure VirtualBox**

#### **1. Configure Clash on the Host Machine**
1. Locate and edit the Clash configuration file (usually `config.yaml`).
2. Enable LAN access by setting:
   ```yaml
   allow-lan: true
   ```
3. Note the Clash LAN IP and proxy ports:
   - Default HTTP Proxy: `198.18.0.1:7890`
   - Default SOCKS Proxy: `198.18.0.1:7891`
4. Restart Clash to apply changes.

#### **2. Configure VirtualBox Networking**
1. Open VirtualBox and select the VM.
2. Go to **Settings > Network**:
   - Set **Adapter 1** to **Bridged Adapter**.
   - Select the appropriate network adapter connected to the internet (e.g., Wi-Fi or Ethernet).
3. Save the settings and start the VM.

#### **3. Set Up Proxy in the Guest VM**
In the guest operating system (e.g., Kali Linux):

##### **Set Environment Variables for Proxy**
2. Get Host Machine's IP Address

To ensure the VirtualBox VM can communicate with the host's Clash instance, you need to know the host machine's IP address:

### Windows:

Open Command Prompt and run:

```ipconfig```

Look for the active network connection (e.g., Wi-Fi or Ethernet). Note the IPv4 address, such as 192.168.0.105.

### Linux/macOS:

Open a terminal and run:

```ifconfig```

or

```ip a```

Note the IP address of the active network interface, such as 192.168.0.105.

1. Open a terminal and edit the `/etc/environment` file:
   ```bash
   sudo nano /etc/environment
   ```
2. Add the following lines:
   ```bash
   export http_proxy="http://198.18.0.1:7890"
   export https_proxy="http://198.18.0.1:7890"
   export no_proxy="localhost,127.0.0.1"
   ```
3. Save and close the file.

##### **Apply Proxy for APT (Optional)**
1. Create or edit the file `/etc/apt/apt.conf.d/95proxies`:
   ```bash
   sudo nano /etc/apt/apt.conf.d/95proxies
   ```
2. Add the following lines:
   ```bash
   Acquire::http::Proxy "http://198.18.0.1:7890";
   Acquire::https::Proxy "http://198.18.0.1:7890";
   ```
3. Save and close the file.

##### **Reload the Environment**
1. Reload the proxy settings:
   ```bash
   source /etc/environment
   ```

#### **4. Test the Configuration**
1. Verify connectivity with the proxy:
   ```bash
   curl -x http://198.18.0.1:7890 https://ifconfig.me
   ```
   This should return the public IP address as seen through Clash.

2. Test DNS resolution and web access in the browser. Ensure the browser is configured to use the system proxy or explicitly set to the Clash proxy.

---

### **Advanced Configuration: Transparent Proxying**
To route all traffic (not just HTTP/HTTPS) from the VM through Clash:

1. **Enable Clash TUN Mode**:
   - Edit the Clash configuration file:
     ```yaml
     tun:
       enable: true
       stack: system
       auto-route: true
     ```
   - Restart Clash.

2. **Set the Host as the Default Gateway in the VM**:
   ```bash
   sudo ip route add default via 198.18.0.1
   ```

3. Test the configuration by checking the public IP for all traffic:
   ```bash
   curl https://ifconfig.me
   ```

---

### **Troubleshooting**
- **Issue: Proxy Not Working**
  - Verify Clash is running and `allow-lan` is enabled.
  - Check if the VM can ping the host IP (e.g., `ping 198.18.0.1`).

- **Issue: SSL Errors in the Browser**
  - Import the Clash CA certificate into the VM’s trusted certificates.
  - For Linux:
    ```bash
    sudo cp clash-cert.crt /usr/local/share/ca-certificates/
    sudo update-ca-certificates
    ```

- **Issue: DNS Resolution Fails**
  - Use public DNS servers in the VM by editing `/etc/resolv.conf`:
    ```bash
    nameserver 8.8.8.8
    nameserver 8.8.4.4
    ```
  **Some trouble shooting links**
  https://medium.com/@4xpl0r3r/deal-with-the-network-issue-of-udp-services-with-clash-tun-mode-enabled-274cca074f90
---

### **Conclusion**
By following these steps, you can configure your VirtualBox VM to route its traffic through the Clash proxy running on the host machine. This setup provides enhanced control over the VM’s network traffic and allows for seamless integration with Clash’s powerful proxying capabilities.



