# How to configure your UniFi network for Sonos

## Goal

Using [Sonos](http://www.sonos.com) devices in a [UniFi](https://www.ui.com/consoles/) network has been a common source of network issues which can be hard to detect. Over the years, there have been many threads in the [UniFi Community](https://community.ui.com), [Sonos Community](https://en.community.sonos.com), Reddit, and other places. This document aims to provide a canonical, community-driven, and up-to-date reference for the most common use cases.

## Symptoms

If you have Sonos devices in your UniFi network, you may experience some of the following symptoms which may appear unrelated but are a consequence of broadcast storms:

- .local name resolution not working
- Sonos speakers disappear from the network
- AirPrint-capable printers are not available
- HomeKit devices not found
- Time Machine backups fail
- Mobile applications fails to discover devices on the local network (e.g. the IKEA Smart Home app)

## Root Cause

Sonos OS (even the current S2) [uses older / pre-standard STP path costs](https://en.community.sonos.com/advanced-setups-229000/will-sonos-s2-support-rstp-or-newer-stp-path-costs-6841084) which makes it incompatible with the newer [RSTP protocol](https://en.wikipedia.org/wiki/Spanning_Tree_Protocol#Rapid_Spanning_Tree_Protocol) which was introduced in 2001 and is the default for UniFi switches. STP can take up to a minute to converge, while RSTP typically converges under ten seconds in normal operation.

This becomes a problem when you operate both wired and wireless Sonos device in your network (even without SonosNet) and can result in a wireless path being preferred over wired, ports being blocked, or broadcast storms.

## Recommended Settings

Ubiquiti has published best practices on https://help.ui.com/hc/en-us/articles/18930473041047-Best-Practices-for-Sonos-Devices. These are also linked to from the detail pages of Sonos devices in the Network application.

The best practices are unfortunately quite restrictive and suggest that once there is a single wireless Sonos device on the network, no other Sonos device may be wired.

You can use the following settings (as of Sonos OS S2 13.2, UniFi Network Application 8.3.32) to mix wired an wireless Sonos devices:

- IoT Auto-Discovery (mDNS): _on_ (likely required only if Sonos devices are segregated into a separate VLAN)
  - Settings -> Networks
- Multicast Filtering (IGMP Snooping): _on_ (helps reduce the multicast traffic from Sonos devices)
  - Settings -> Networks
- Multicast Enhancement (IGMPv3): _on_
  - Settings -> Wireless Networks -> `$YOUR_NETWORK`
- Multicast and Broadcast Control: _off_
  - Settings -> Wireless Networks -> `$YOUR_NETWORK`
- Spanning Tree: _RSTP_
  - Settings -> Networks -> Global Switch Settings
  - UniFi Devices -> `$DEVICE` -> Settings -> Advanced -> Spanning Tree Protocol (for switches where Global Switch Settings is disabled)
- Port settings on the LAN ports that connect to Sonos gear: disable _Spanning Tree Protocol_
  - UniFi Devices -> `$DEVICE` -> Port Manager -> `$AFFECTED_PORT` -> Advanced

Alternatively, you can change all your switches to use STP instead of RSTP, but this may make acquiring an IP over DHCP slow. If you do that, assign priority values manually: use priority 4096 for your main switch (going to your router/firewall) and add 4096 for each hop from there (e.g. 8192 for second-level switches, 12288 for third-layer switches). However, the [UDM, UDMP, UDM-SE, and UDM-Pro-Max do not support STP](https://community.ui.com/questions/UDM-Pro-Ability-to-Toggle-from-RTSP-to-STP/45c8751b-2611-4e78-a779-6846b2dbb9a2). It can be temporarily enabled using `brctl setbridgeprio br0 4096; brctl stp br0 on` (or just don't connect any wired Sonos devices to the internal switch of a UDM).

On the Sonos side, SonosNet should be disabled. This can be achieved by disabling WiFi on every wired Sonos device.

## References

### Official Documentation
- [Best Practices for Sonos Devices](https://help.ui.com/hc/en-us/articles/18930473041047-Best-Practices-for-Sonos-Devices)
- [Configure STP settings to work with Sonos](https://support.sonos.com/s/article/2118?language=en_US)
- [UniFi - USW: Configuring Spanning Tree Protocol](https://help.ui.com/hc/en-us/articles/360006836773-UniFi-USW-Configuring-Spanning-Tree-Protocol)

### Community Threads
- [Issues with mDNS / multicast / AirPrint / HomeKit / Sonos](https://community.ui.com/questions/Issues-with-mDNS-multicast-AirPrint-HomeKit-Sonos/3d103b00-f1fa-40e0-8c49-ec0a0121e93c)
- [Will Sonos S2 support RSTP or newer STP path costs?](https://en.community.sonos.com/advanced-setups-229000/will-sonos-s2-support-rstp-or-newer-stp-path-costs-6841084)
- [Sonos and Unifi gear / VLANs - RSTP update](https://en.community.sonos.com/advanced-setups-229000/sonos-and-unifi-gear-vlans-rstp-update-6830571)
- [UniFi, STP and Sonos](https://community.ui.com/questions/UniFi-STP-and-Sonos/7f72d9cf-6511-42f6-b6bc-d9b5efb7cb19)
- [Google Home speaker groups/Homekit/Sonos/Airprint/IoT/Multicast/mDNS issues?](https://community.ui.com/questions/Google-Home-speaker-groups-Homekit-Sonos-Airprint-IoT-Multicast-mDNS-issues/294320bd-be6d-4745-b74c-eba70f40958c)
- [HomeKit Multicast issues across a wireless mesh link](https://community.ui.com/questions/HomeKit-Multicast-issues-across-a-wireless-mesh-link/106e0fca-b10a-42b5-9100-4848719e8b84)

### Other

- [Configure STP settings to work with Sonos](https://support.sonos.com/en-us/article/configure-stp-settings-to-work-with-sonos)
- [Overcoming Sonos Issues with Araknis Networks Equipment](https://www.snapav.com/wcsstore/ExtendedSitesCatalogAssetStore/attachments/documents/Networking/SupportDocuments/Sonos_TSB.pdf)
