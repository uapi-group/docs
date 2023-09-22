# ﻿Image-Based Linux Summit Berlin 12th September 2023

## Attendee's projects
* Systemd
* Meta: mkosi, systemd, remote attestation
* SUSE: Aeon/MicroOS/Tumbleweed
* Redhat: image-builder/osbuild, systemd, systemd-boot
* Microsoft: confidential containers, flatcar, Azure Boost, Mariner
* Edgeless Systems: Constellation, confidential containers
* NixOS: systemd-{repart,stub,boot,cryptenroll}, Tvix Store, Lanzaboote, UKIs, NixOS Bootspec,, PE binary manipulation in Rust
* Canonical: Ubuntu Core, snaps, foundations/bootloader/package management
* System Transparency: UKI, UAPI specs, systemd-cryptenroll, remote attestation, TPM eventlog, mkosi
* Amazon Linux
* Freedesktop-sdk
* GNOME OS
* carbonOS

## Summary of Last Year’s Progress
* UAPI Group established
* Website! https://uapi-group.org 
* Github org https://github.com/uapi-group
* Report on previous summit: https://lwn.net/Articles/912774/
* Systemd:
   * systemd-boot/stub: addons support (only cmdline, dtb incoming)
   * Systemd-stub: confidential VM support
   * UKI: new sections in the spec for metadata (uname/sbat)
   * Ukify: build UKIs, embed measurements and signatures
      * Calls systemd-measure, sbsigntool
   * Mkosi now supports UKI directly, building of initrds
      * Uses systemd-repart, unprivileged, can use from containers, UKIs support with PCR measurements bells and whistles, embed OS inside UKI (small img), massive refactoring, broke everything and fixed again (?), presets (multi-image layers in same build, as dependencies for sequential stages of build), initrd built directly without dracut, almost reproducible (WIP), improvements around secure boot signing, supports grubs (unprivileged, shim works if it is installed to /boot), credentials support in cfg file
   * Systemd-measure: PCR11 measurements and signing for UKI
   * Pcrphases: advances the PCR during boot (initrd -> rootfs transition)
   * Kernel-install is C and has plugins for UKIs
   * Systemd-repart: running unprivileged, minimization of filesystem, support dm-verity, reproducibility (repart itself, but mkfs.* might not), direct writing of file system contents, subvolumes for btrfs (needs privileges)
   * Bootctl: garbage collection of /boot (bootctl unlink), delete kernels that are not referenced, bootctl cleanup for other files as well, bootctl identify/inspect
   * Early-boot detection of battery critical levels
   * Smbios support: pass kernel cmdline, credentials
   * AF_VSOCK support in systemd: sd-notify support to pass back state changes to hypervisor (ready, failed, finished)
   * Credentials: almost everything that takes config files takes credentials (no udev, no units)
      * Can now process credentials from systemd generators
   * RPM gained native support for sysusers
   * Softreboot
      * Skip kernel restart, jump to new rootfs
      * Pass state across via FD Store, retains /run
      * Can configure some services to survive
   * TPM: offline sealing
      * Get TPM public key, provision secrets from different machine
      * Confidential computing support: setup SRK before VM instantiation, using to seal secrets
      * Fully encrypted sessions, pin SRK, create SRK if not present already
   * Confext: equivalent of sysext (system extension) but /etc
* Grub: patch sent for UKI support, under review
   * AI Luca: enquire about state
* NixOS:
   * Working on Rust version of systemd-stub
   * Uses repart for DDI
   * Bootspec: high level config to support multiple bootloaders
   * Rust implementation of ukify: goal to do everything in-process without shelling out
   * Systemd-sysupdate + repart integration
   * tvix-store for serving content addressable parts of (system) images: some sort of blob protocol relying on Merkle trees / BLAKE3 constructions
   * We have a systemd-based initrd implementation finally with networkd
* Flatcar
   * Uses systemd-sysext for A/B updated OEM software (cloud vendor / virtualisation guest tools)
   * sysext-bakery gives simple scripts for users to build custom sysext images, prebuilt images published as github artifacts with systemd-sysupdate config (signing WIP), sysupdate can consume them
   * flatcar-reset tool: stage reset on next reboot, with optional regex to keep files around
   * /etc is now overlay mount, lowerdir is on read-only /usr/share/flatcar/etc, /etc remains writable (/etc is upperdir, unmounting overlayfs still results in a working system, needed to suppress a few tmpfile rules to prevent upcopies)
* OSBuild
   * Support for UKI (native implementation)
* Ubuntu: TPM support by default on generic distro
   * Snapased: kernel updates, measurement updates
   * Uses systemd-stub
   * Systemd-based initrd in Ubuntu-core
   * Ubuntu-core initrd+kernel snap used for tpm-by-default desktop
   * Native upstream cryptsetup
   * Precalculate PCR measurements
   * White paper internal, not published yet
* GNOME OS:
   * Sysupdate support, already used sd-boot
   * Repart on firstboot
   * UKI support, with dracut
* SteamOS supports sysext too now
* SUSE
   * Shim trusts signed systemd-boot in openSUSE
   * YaST support to install with systemd-boot in MicroOS (tooling works on Tumbleweed too)
   * PCR Oracle tool for PCR predictions
   * Aeon (formerly micro-os desktop), sd-boot, image-base-install, full FDE by default no option
   * Tumbleed installs everything into /usr (no longer /boot) work in progress on /etc too
* Anaconda installer
   * Supports sd-boot

## Topics
### Not covered
* Integrity of OS images, via Verity and so on.
* Updating OSes safely and robustly in A/B fashion
* Booting OSes in A/B fashion
* Provisioning image-based OSes with credentials, parameters and similar
* Signed kernels, root fs + SecureBoot
* Hermetic /usr/ + Hermetic /etc/
* TPM provisioning, enrolling, measurement, remote attestation
* Composing/Modularity of OSes based on multiple images
* Build systems for OS images
* Version management, rollback control
* Vendor/User Resource isolation
* Reproducibility of the build process
* Bootstrapability of build process (https://bootstrappable.org/)
* Secure configuration management (signed configuration updates)
* OS Image formats
* Unified kernels + pre-built initrd
* Disk encryption in images
* Novel Linux file system technology for immutable OS images (blobfs, …)
* OS Factory Reset
* Systemd-pcrlock
* TPM event log
* Securely pass cmdline to UEFI via UKI (we deprecated SystemdOptions UEFI var recently)
* Shim
* How to split up root vs home partition space
   * Use OPAL

### Covered
* Sysext / confext and mutable /etc, /usr, and /opt (https://github.com/flatcar/Flatcar/issues/986#issuecomment-1571429705 , https://github.com/systemd/systemd/issues/24864#issuecomment-1576792335 )
   * Currently sysext/confext overlay is R/O
   * Can we add a writable layer/mode (not default)
   * R/W can be ephemeral or persistent
   * OS layer at the top? Right now at the bottom of the stack
   * Fix it by symlinks! Always the answer
   * AI: add ordering guarantee to sysext spec
   * AI: rewrite config spec on uapi-group to remove XDG duplication
   * Ideally: overlayfs mode that allows creating new files, but not modify existing files from other layers
   * AI: Propose an extension to sysext and confext specs on uapi-group for mutable and ephemeral modes.
* Hermetic /etc
   * SUSE adds support for libeconf in upstream projects
   * apache2/nginx still need files in /etc to work, part of openssh, complex configurations files (xml or so)
   * Fedora also has patches
* /efi vs /boot vs /boot/efi vs /run/efi
   * filesystem on SUSE is rpm that creates base directories
      * Base-files on Debian/Ubuntu
      * Top-level directories are problematic as mountpoint for this
      * /run ? runtime object, but solves rpm problem, also there are assumptions in systemd about resources /run
         * Offline mounting also problematic, tools assume /run is not there before system is booted, ordering problem
         * /run mounted by container manager/init
   * Ideally, use automount
      * Why do we even mount it? Because: bootctl/fwupd/kernel-install/logind(r/o)
      * Ideally only one tool touches bootloader after all updates are applied to disk
   * Xbootldr partition vs esp, if both available then xbootldr needs go to /boot
      * The most important thing is where to put kernels
      * So if xbootldr partition, then mount it on /boot
      * If no xbootldr then ESP to /boot
      * So kernels are always installed to /boot
      * Places to put bootloader is not stable
      * Bind mount or symlink, but implementers of bls need to understand this
      * Spec for BLS: https://uapi-group.org/specifications/specs/boot_loader_specification/#mount-points
         * Conflicts with Discoverable Partitions Spec: https://uapi-group.org/specifications/specs/discoverable_partitions_specification/ (explanation for ESP GUID table)
         * DPS says default is /boot, BLS says /efi
         * Tools do it the DPS way
         * Fix BLS? /boot as default
         * https://github.com/uapi-group/specifications/issues/35
         * AI: send PR and merge
         * Tools shouldn’t mount twice
      * /run/efi can be mount/bind mount/symlink to ESP?
      * APIs
         * bootctl -p or -x as API to find location of ESP
         * Bootupd as API for grub support for BLS in Fedora
         * Do we need new API?
         * /dev/-by symlinks? Install kernels ‘here’, bootloader ‘here’
         * Still need to dedup in readers
         * bootctl/kernel-install as multicall that can be called or provide service with varlink, works inherently with –image/–root
         * Bootctl-varlink.socket
         * AI Luca: open RFE on github.com/systemd/systemd
      * Sd-boot recognizes what an EFI binary is via .sdmagic
      * Dual boot causes a lot of problems
         * Makes our lives complicated, do we really need it?
         * Developers need it for hw-related development
         * Not supposed to measure efi entries that are tried, only the one actually booted
         * Windows: set BootNext and reboot
         * Sd-boot supports it but only in ‘bitlocker mode’, should be made generic when switching OS
         * UEFI has boot counter in the spec, so writing to variables should be fine
* Talk about AWS/Azure/GCP IMDS (early-boot link-local networking metadata server) and credentials / generators / network config
   * Some previous discussion between Arian/Lennart: https://gist.github.com/arianvp/22e1c5182eb6c17bbd8c1bbe823b516b
   * Networkd cloud-provider-based configuration: https://github.com/systemd/systemd/issues/16547
* VBE: https://u-boot.readthedocs.io/en/latest/develop/vbe.html — alternative to UEFI for embedded systems, maybe more?
   * NixOS looking into this, for coreboot support
   * Specs covers boot flows, OS requests
   * Interoperates with UEFI
   * Other firmwares: tianocore/edk2, slimbootloader, LinuxBoot, u-boot, coreboot, SeaBIOS
   * Emulate uefi on bios
      * Nix has some shims to support UEFI variables
      * Uboot can be build as bios payload, with UEFI upper layer, store variables in TPE or text files on boot partition or just leave everything volatile
      * Google has KVM solution
      * Clover EFI bootloader for BIOS, see configuration
      * Question: is there a UEFI protocol to detect that persistent vars are actually ephemeral?
      * Fedora: proposal to drop bios support, but protests, 30% of machines bios-only, still hybrid
      * Amazon Linux 2023 has dual bios/uefi hybrid support
      * Hybrid images: tricky to support everything because of 512/2048/4096 partition headers. mkosi tried but gave in on El Torito support.
* Provisioning systems for VMs
   * SMBIOS support in sd-boot/stub
   * Alternative: ACPI device that can send notifications
      * SMBIOS technically is static, although one field can change at least: suspend/resume field (int) for reason of change
      * Also data might not be ready when guest is booted
   * SMBIOS is already supported widely
   * ACPI needs a driver
   * Requirement: userspace needs to ask for data and wait synchronously, don’t want to boot and then data appears later
   * SMBIOS guarantees that data is already there
   * ACPI can generate data on the fly when requested
   * Systemd provides point at boot where we require data, hypervisor needs to provide it
   * Dracut has pre-existing infrastructure on this topic
   * Sd-stub already supports picking up resources from ESP, also reads smbios
   * New transaction in initrd, units that generate other units
      * Also needed to know if /root is tmpfs in installer image
      * New targets where to hook the metadata
   * Network namespace + VRF for cloud-init and similar to get provisioning info
   * Amazon: provision user’s UEFI variable database
   * Addon: PE non-code binary that can be verified by shim/uefi, append kernel command lines
   * Credentials can be provisioned from /run in initrd to rootfs transition
      * At the new mid-initrd target, provisioning tools prepare creds in /run
      * Then systemd reloads and does second transition
* Credentials
   * Provision SSL cert via creds
   * Need to update it
   * How to do it at runtime?
   * If done live, atomicity is lost
   * Restart service? Use FD store to keep sockets and resources order
   * We will need conftext live reload
   * Mount “beneath” to replace mount point atomically
   * Can do the same for creds?
   * We have the semantics for systemctl ‘reload-or-restart’ already
   * We do not have good docs on fdstore
   * Do we want an API for “Hey what services depend on what credentials so I can restart them”
   * Future improvement: systemd-creds can encrypt to asymmetric TPM SRK so that creds can only read
   * Let's document $CREDENTIALS_DIRECTORY as static for system services.by confidential VM
   * API to create/enumerate credentials is missing
      * Sd-creds as multi-call binary with varlink service
   * Problem: reading creds requires looking at env var, but software doesn’t like the interpolation
   * AI Arianvp: change doc to document that creds path for system service is fixed with precise caveats and warnings to avoid being it misused
   * sd-ask-pwd: no solution for user services
      * More complex issue, needs integration with desktop managers
         * Tracking issue: https://gitlab.gnome.org/GNOME/gnome-shell/-/issues/6921
      * Kernel keyring has advanced lately, very complex
* TPM stuff
   * Currently involves PCR11 (kernel/uki) and PCR15 (system identity)
   * How to cover PCR 0-7 which are owned by firmware?
   * Answer: systemd-pcrlock
   * Instead of changing the superblock on fw update, change the policy in the TPM
   * Attach objects to the TPM policy, when changes are expected change policy objects
   * Disk encryption binds to policies, not measurements
   * .pcrlock files define component involved in boot, even after fw -> kernel transition, json format following TCG measurement log format (but not 100% the same)
   * Allows drop-ins and alternatives for components (eg: run multiple kernels)
   * Generates lock files from what system measured at some point via tpm log
   * If fwupd shipped the pcrlock file with an update then we wouldn’t need to measure on-the-fly, current LVFS stores result of pcr0, instead store measurement
   * Only vendor can know what firmware will measure, so vendor has to provide it
   * Pcrlock tool orders drop-ins by filenames
   * If fwupd doesn’t know what measurements are going to be, it can remove the pcrlock drop-ins that corresponds to the component
   * Pcrlock will recognize that there are holes, and remove that pcr/measurement from the policy
   * If info is there, secrets are always protected
   * If info is not there, secrets are not protected by those pcrs for one boot
   * Improvements: reboot twice to apply changed quickly and minimize window
   * Windows also reboots twice on unexpected fw update, but changes block device and re-enrolls in Bitlocker
   * Can lock against firmware, can deal with firmware updates, if vendors collaborates no ‘open’ policy window
   * Local/hw specific vs OS vendor (uki)
   * Also solves rollback protection
   * Recalculate automatically on reboot so that it always uses the most secure policy possible
   * Can generate from gpt, uki, pe, current event log
   * Trust-on-first-use model
   * Transparent log database with pcr values of known hardware - if anything changes in the uefi (option) they change - from TCG
   * How to recover from failure - recovery key? How to restore a usable policy? Reset TPM?
      * Can the same recovery key for LUKS2 be used?
   * Ubuntu Core also implements a similar infrastructure
      * Changes the LUKS superblock on updates
      * Can be analyzed offline
   * Systemd has event log, that works like TCG firmware event log, for measurements done at runtime, v255
      * Json-seq, not array but separator-separated blocks so that it can be append-only
      * Stored in systemd-specific dir so that it’s private
      * Would be good to allow other usespace app can append
      * But need atomic behaviour (file lock, append-only)
      * Public API dbus/varlink
      * AI Lennart: libsystemd sd_measure(⋅) which takes data and hash it for the consumer
   * Kexec(/crashkernel): measure nonce so that it cannot be predicted and change policy, so that it works only on that kexec
   * Ubuntu Core stores encrypted object that allows to reset the TPM on factory reset
      * Systemd should have a hook for this
      * Ubuntu core code: https://github.com/snapcore/secboot
   * Non-disk-encryption usage of TPM
      * VMs in GCP use TPM for remote attestation on container-optimized linux (fork of chromeos), not on generic linux distro, precalculate expected PCRs
      * System Transparency (https://www.system-transparency.org/ ): remote attestation oss project. Code https://git.glasklar.is/system-transparency/core
      * Attested TLS: developed for SGX - standardization happening, RATS, remote attestation for TLS
      * Bind confidential computing to remote attestation
      * Keylime uses TPM for remote attestation, integrated in MicroOS
      * Talos does <something>
      * Build reproducibility helps for build systems attestation, but it is expensive, so many prefer attestation/authentication as an alternative
   * Confidential computing
      * Amd sev vs intel tdx, amd uses tpm
      * Full vm confidential comp vs kata containers, but still needs measured boot
      * In Azure firmware does measurements in vTPM
      * Thread models driven by soc - intel/amd - no common specification
* Unified kernels + pre-built initrd
   * Pass initrd via PE binary, lose ordering guarantees because filenames are not trusted
   * Dtb addon patch adds matching based on content of dtb
   * .mucode for microcode section in UKI, and magically make it first in the generated cpio
* Systemd-sysusers and user users
   * A switch will be added to copy /etc/skel
   * Can also drop json blob for userdbd to pick up in /run/ for transient user user, never show up in /etc/passwd → also great for hermetic /usr with json blobs in /usr/lib/userdb/
   * pam_mkhomedir can also create the directory on first login
* Homed in openSUSE Aeon: many papercuts, mainly storage burden
   * Need to know a lot of stuff for provisioning ahead of time (like number of users)
   * Encrypted btrfs subvolume as a solution, don’t need to know size ahead of time
   * Lots of work going on in Gnome, lock session and remove disk keys from memory
   * Interactive resizing negotiating between users if disk needs to change (owner user will be authenticated and asked to agree)
   * SELinux policy for homed needs a little work
   * Fedora actively fixes systemd selinux issues if they are reported with logs
   * Some users in Nix also use homed, but with fixed user, also on Fedora there are some users but unofficial
   * Ongoing work in Gnome is to make it trivial to enable homed in distros
   * How to split space between /home and rootfs
      * New partition type in repart using dm-linear to add an extension for another partition on-the-fly
      * Android uses it, no measurable performance penalties for having many dm-linear extensions
      * / as opal encrypted (cryptsetup will support in next version with LUKS2 for keys), then homed LUKS2+dm-crypt loopback
      * q devices
      * Homed could use creds for initial setup
         * Sysusers runs much earlier, homed need a lot of stuff set up, runs very late
   * Why not just use ZFS?

## Action Items
* allow opt-in creation of “normal” users via sysusers
(Possibly use pam module to create the directory on first login?)
* bootctl -X (name TBD) to print all paths containing boot loader entries[a][b]
* bootctl status --json
* Link sd-boot BootNext generalization github ticket   https://github.com/systemd/systemd/issues/22390
* Arian sends documentation PR about credential path for system services
* AI: fix incompatibility w.r.t. Mount points between BLS and DPS
* AI: ask shim to cap sd-stub’s PCRs (os vendor name or so)
* AI: start issue or so to track hermetic-usr work-in-progress
* AI: create spec for drop-in style config management
* AI: add ordering guarantee to sysext spec
* AI: Propose an extension to sysext and confext specs on uapi-group for mutable and ephemeral modes.

