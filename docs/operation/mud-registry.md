## Operations guide for the Micronets MUD Registry
### for NCCoE Build 3

The MUD Registry provides an HTTP API used to manage the association of MUD files with devices. Available operations:

- Register a device model using:
  + A public key as an unique device identifier
  + A vendor code to identify a vendor/manufacturer
The supplied device model is used as the MUD file name. The MUD file is located relative to a base URL as configured by the MUD registry.

- Locate or return a MUD file using:
  + A vendor code and a public key
  + A vendor code and a device model

- Deregister a device public key (removes the pubkey->device model relationship)
- List all registered pubkey->device model relationships
- Locate the base URL of a vendor registry using a vendor

All of these requests can be submitted to a global registry, providing a vendor code to facilitate redirection, or directly to the vendor endpoint.

[MUD Registry Operations Guide](https://github.com/cablelabs/micronets-mud-registry/blob/nccoe-build-3/README.md#Operation)
