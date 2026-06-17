# Deployed-Wireless-Infrastructure-with-CAPsMAN-Hotspot-RADIUS-and-VRRP-Failover
### *Engineered high-availability wireless infrastructure using MikroTik CAPsMAN for a central wireless access point management and stateful Hotspot captive portals synchronized via external RADIUS authentication over redundant VRRP/MSTP gateways.*

  > My Fourth Project
---

## The Topology

<img width="1487" height="860" alt="TOPOLOGY3" src="https://github.com/user-attachments/assets/1f392ad2-03d2-4802-9674-264b51443dfd" />

---

## The Lore

Okay so here's the scenario:

Anyone within range can see your SSIDs. Anyone can try to connect. And guests? They need internet access, but they can't touch internal resources.

I needed wireless that was enterprise-ready:

- **Users vs Guests:** Two SSIDs. Two VLANs. Zero crossing.
- **Hotspot Portal:** Guests authenticate via captive portal, not just a shared PSK
- **RADIUS Authentication:** Centralized credentials via dedicated User Manager router
- **Controller Redundancy:** If the primary CAPsMAN dies, the backup takes over
- **Gateway Failover:** VRRP keeps default gateways alive during core failure

The challenge? Getting all these systems to play nice during failures. CAPsMAN failover. Hotspot state. RADIUS continuity. VRRP migration. Each has its own timeout. Each can break the others.

I used a dedicated MikroTik router as my external RADIUS server running User Manager. Both core routers point to it for authentication. Simple. Centralized.

Here's what happened.

---

## Wireless Architecture

### CAPsMAN Centralized Management

**What I did:** Deployed CAPsMAN a central wireless access point manager on both core routers redundantly. Created two SSID profiles:
- **Users SSID:** VLAN 10
- **Guests SSID:** VLAN 20

Pushed configurations to all CAP devices. One master controller. Automatic provisioning.

**What it achieves:** No per-AP configuration. Everything managed through CAPsMAN. Consistent setup everywhere.

---

### Hotspot Captive Portal (Guest Network)

**What I did:** Enabled Hotspot on the Guest SSID (VLAN 20). Configured captive portal with custom login page. All guest traffic is blocked until authentication completes.

**Hotspot features deployed:**
- RADIUS authentication against User Manager
- Session accounting and bandwidth limiting

**What it achieves:** Guests don't just get a PSK they can share. They need individual credentials. Vouchers can be time-limited. Access is tracked.

---

### External RADIUS with User Manager

**What I did:** Configured a dedicated MikroTik router as an external RADIUS server running User Manager (IP: 192.168.1.3). Both core routers point to this single RADIUS endpoint for:
- CAPsMAN user authentication (WPA2-Enterprise)
- Hotspot user authentication (Captive Portal)
- Vouchers

**What it achieves:** Centralized credentials. One source of truth. Both CAPsMAN controllers use the same authentication backend, so failover doesn't break login. Hotspot sessions persist across gateway failover.

---

### VRRP Gateway Redundancy

**What I did:** Deployed VRRP across both core routers for VLAN 10 and VLAN 20. Virtual gateway IP moves seamlessly between cores. Client default gateway never changes, even when the active router dies.

**What it achieves:** Client sessions survive core router failure. Hotspot sessions remain active. No manual reconfiguration.

---

## The Proof

### Test 1: Provisioning SSID Broadcast and Automatic VLAN Tagging

**What I did:** Powered on both CAPs. Watched them join CAPsMAN automatically. Scanned for SSIDs.

**What happened:** Both Acess Points appeared in CAPsMAN. SSIDs "Users" and "Guests" visible. AP1 on Channel 1 (2412 MHz) . AP2 on Channel 6 (2437 MHz). 
No manual config on each CAPs (Access Points). Users SSID got IP from VLAN 10. Guests SSID got IP from VLAN 20. Traffic isolated at Layer 2. Zero-touch AP deployment.


https://github.com/user-attachments/assets/cf41f4c8-7729-4382-a6e5-de0760f9031d


---

### Test 2: Hotspot Captive Portal Authentication

**What I did:** Connected to Users and Guests SSID (open network). Opened browser. Entered Hotspot credentials.

**What happened:** Browser redirected to captive portal. Hotspot authenticated. Traffic flowed.

https://github.com/user-attachments/assets/d821f5fd-8065-4e4b-8be2-490566accd6b

---

### Test 3: Hotspot Redirection with Centralized Radius Authentication

**What I did:** Connected to Guests & Users SSID and opened the captive portal. Entered active credentials verified by the external User Manager server backend.

**What happened:** Hotspot offloaded the handshake to the RADIUS backend over port 3799. Radius Server verified the lease, and dynamically authorized the session.


https://github.com/user-attachments/assets/c54f0d66-923a-480d-a848-e828d6770322

---

### Test 4: CAPsMAN Controller with VRRP Failover

**What I did:** Took down Core R1 while client held Hotspot session.

**What happened:** CAP detected primary loss within 3 seconds. Established new tunnel to Core R2 in ~6 seconds. Full CAP re-join at ~9.2 seconds. Hotspot config re-applied within ~15-20 seconds. Wireless recovers, due to automatic session Cookies. No re-authentication needed.

https://github.com/user-attachments/assets/96d21ee4-f8b8-4a5d-80a0-0d8f5003c68d

---

## Challenges I Faced

**Hotspot Session State During Failover**

When the primary CAPsMAN fails, the secondary controller doesn't automatically know which Hotspot sessions were active.

**How I solved it:** Kept RADIUS on a dedicated external router and enabled Cookies. When the secondary takes over, it re-validates active sessions against RADIUS. Users don't get kicked to login page again.

**RADIUS Server Independence**

The architectural win was keeping RADIUS on a dedicated router, not colocated on the CAPsMAN controllers. When a core router fails, Hotspot authentication still works.

**What I learned:** Don't bundle services. Separate authentication from access.

**RADIUS Port 3799**

User Manager uses port 3799 for RADIUS, not standard 1812. Almost missed this during configuration.

---

## Final Thoughts

Wireless doesn't have to be the weakest link.

I built a wireless network that:
- Auto-provisions CAPs with zero-touch config
- Segregates users and guests via VLAN tagging
- Forces guests through Hotspot captive portal
- Authenticates everyone against central RADIUS
- Survives core router failure without re-login
- Recovers controller failure within ~20 seconds

---

## Future Enhancements

- **RADIUS MAC authentication for VIPs:** Auto connect via hardware addresses, bypass captive portal
- **IoT IP bindings:** Exclude headless devices (CCTV, printers, IP phones) to skip portal redirection
- **CAPsMAN certificates:** Generate certificate and require the Access points to only connect when it has one
- **Let's Encrypt SSL:** Encrypt and map portal to subdomain (e.g., login.company.com) with auto-renewal
- **Dynamic walled gardens:** Support HTTP-CHAP/PAP, and redirect clients to login page
- **Custom HTML5 portal:** Responsive and customized ad ready branded pages

---

*Fourth project down. More to come.* 🔥
