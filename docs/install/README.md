## Installation guide for Micronets components

### for NCCOE Build 3

This guide describes the installation of the Micronets
cloud components on a cloud server, enable it for
connecting Micronets gateways, setup of the MUD Manager
and MUD Registry, and provisioning devices using the 
Micronets onboarding application and WiFi EasyConnect/DPP.

Prerequisites:

* A public server (for cloud components) with:
   * Ubuntu 18.04 LTS
   * A registered DNS name
   * A web/HTTPS certificate associated with the registered DNS name
   * 4GB of RAM
   * 50GB of free disk space

* A Linux server connected to your LAN with:
   * Ubuntu 16.04 LTS
   * A wireless adapter compatible with hostapd with DPP support
   * Software requirements documented here

High-level Steps:

0. [Install & configure the Websocket Proxy service](ws-proxy.md)
0. [Install & configure the MSO Portal service](mso-portal.md)
0. [Install & configure the MUD Manager service](mud-manager.md)
0. [Install & configure the MUD Registry service](mud-registry.md)
0. [Install & configure Micronets Manager](micronets-manager.md)
0. [Example single-server nginx configuration](example-1server-nginx.md)
0. [Install & configure the Micronets mobile app](mobile-device.md)
0. [Setup & configure the Pi test device](pi-test-device.md)

Once the Micronets cloud components are installed the [Micronets
Operation Guide](../operation/README.md) provides a guide for adding/removing subscribers, 
creating/removing Micronets, and onboarding/removing devices.