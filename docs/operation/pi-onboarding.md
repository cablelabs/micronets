## Onboarding a Pi device onto a MICRONET

### for NCCoE Build 3

#### Prerequsites:
- A Raspberry Pi with the `proto-pi` software installed & configured:
  [Installation Guide - Pi](docs/install/pi-test-device.md)
- An iOS or Android phone with the `Micronets` application installed & configured:
  [Installation Guide - Mobile](https://github.com/cablelabs/micronets-mobile/blob/nccoe-build-3/README.md#Installation)
- A Micronets subscriber account (See documentation for Micronets Manager)
- A Gateway device associated with the Micronets subscriber (See documentation for Micronets Gateway)

#### Instructions:
- On first run, ensure an Ethernet cable is attached to the Pi
- Power on the Pi device. The device will automatically be registered on first run
- Power on the Mobile device and log in with your subscriber credentials
- On the Pi, press/tap the Onboard icon. A QRCode appears
- On the Mobile device, tap the "Ready to Scan" button
- Scan the QRCode
- If a MUD file was found, the device **CLASS** and **NAME** will be prepopulated, modify as needed
- Set the **MODE** to STA
- Tap the Onboard button
- Observe the results on the Pi device

#### Further Reading:
[Operation Guide - Pi](https://github.com/cablelabs/micronets-pi3/blob/nccoe-build-3/README.md#Operation)

[Operation Guide - Mobile](https://github.com/cablelabs/micronets-mobile/blob/nccoe-build-3/README.md#Operation)
