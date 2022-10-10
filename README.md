# Generic documents and meeting minutes

This repo contains generic, org-wide documents like meeting minutes.

# Glossary

This section clarifies on terms and abbreviations used in specs and other documents throughout the organisation.

- *DDI*[1] - Discoverable Disk Image
- *DPS*[1] - Discovery Partition Specification
- *MOK* - Machine Owner Key (shim)
- *UKI*[1] - Unified Kernel Images (sd-stub + kernel + initrd + more)
- *BLS*[1] - Boot Loader Specification
- *PCR* – TPM Platform Configuration Registers
- *TPM* – Trusted Platform Module (security chip)
- *sysext*[1] – System Extension Image (type of DDI that is overlayed on top of /usr/ via overlayfs and can extend the underlying OS vendor resources in a composable, immutable fashion)
- *syscfg*[1] – System Configuration Image (type of DDI that is overlayed on top of /etc/ via overlayfs and can extend the underlying OS' configuration in a composable, immutable fashion)
- *SBAT* – UEFI Secure Boot Advanced Targeting

[1] *TODO* add reference to spec as soon as it's up at https://github.com/uapi-group/specifications.
