# Generic documents and meeting minutes

This repo contains generic, org-wide documents like meeting minutes.

# Glossary

This section clarifies on terms and abbreviations used in specs and other documents throughout the organisation.

## General terms and abbreviations
- *MOK* - Machine Owner Key (shim)
- *PCR* – TPM Platform Configuration Registers
- *TPM* – Trusted Platform Module (security chip)
- *SBAT* – UEFI Secure Boot Advanced Targeting

## Terms and abbreviations specific to UAPI group specifications
- *DDI*[1] - Discoverable Disk Image
- *DPS*[1] - Discovery Partition Specification
- *UKI*[1] - Unified Kernel Images (sd-stub + kernel + initrd + more)
- *BLS*[1] - Boot Loader Specification
- *sysext*[1] – System Extension Image (type of DDI that is overlayed on top of `/usr/` and `/opt/` via overlayfs and can extend the underlying OS vendor resources in a composable, immutable fashion)
- *syscfg*[1] – System Configuration Image (type of DDI that is overlayed on top of `/etc/` via overlayfs and can extend the underlying OS' configuration in a composable, immutable fashion)

[1] *TODO* add reference to spec as soon as it's up at https://github.com/uapi-group/specifications.

# Minutes and summaries

Listed below are references to the latest UAPI group meeting / conference summaries and minutes.
See the [minutes](minutes/) folder for a full list of documents.

* Summary of the first Image-Based Linux Summit, October 4th / 5th in Berlin: [Image-based-linux-summit.md](minutes/2022-10-05__Image-based-linux-summit.md)
