# ﻿Image-Based Linux Summit Berlin 24th September 2024

## Attendee's projects
* systemd
* mkosi
* SUSE: MicroOS/Tumbleweed
* Red Hat: [image-builder/osbuild](https://osbuild.org), [bootc](https://github.com/containers/bootc), systemd, systemd-boot
* Microsoft: confidential containers, Flatcar, Azure Boost, Mariner/Azure Linux
* Edgeless Systems: Constellation, [Contrast](https://github.com/edgelesssys/contrast) (confidential containers), [uplosi](https://github.com/edgelesssys/uplosi)
* NixOS: systemd-{repart,stub,boot,cryptenroll}, tvix-store, [Lanzaboote](https://github.com/nix-community/lanzaboote), UKIs, [NixOS Bootspec](https://github.com/NixOS/rfcs/blob/master/rfcs/0125-bootspec.md), PE binary manipulation in Rust
* Canonical: Ubuntu Core, snaps, foundations/bootloader/package management
* System Transparency: UKI, UAPI specs, systemd-cryptenroll, remote attestation, TPM eventlog, mkosi
* Freedesktop-sdk
* GNOME OS
* Pengutronix: OE/Yocto, nspawn, RAUC, verity, OPTEE
* Arch Linux: sbctl/go-uefi, signstar, mkinitcpio, arch-install-scripts
* [PostmarketOS](https://postmarketos.org/)

## Summary of Last Year’s Progress

* Flatcar:
  * Sysexts widely adopted: In-system (shipped with OS), optional but system dependent (updated in lock-step with OS, like podman/ZFS), Custom / user provided (updated from custom servers via sysupdate)
  * Composable OS is more flexible, some traction with contributors, even vendors (Akamai)
  * Mutable sysexts feature available in systemd-256 and newer. Slowly trickling into distros.
  * OS test suite keeps finding bugs in LTS kernel patch-level updates 😅
* NixOS:
  * Systemd-repart integration to build NixOS images
    * Recent dm-verity support for `/nix/store` (incl. offline Secure Boot signing support)
    * UKI available by default
    * [https://github.com/nikstur/userborn](https://github.com/nikstur/userborn) like sysusers but can create “normal” users and “remove” users
    * [https://github.com/nikstur/nix-store-veritysetup-generator](https://github.com/nikstur/nix-store-veritysetup-generator) for `storehash`
* Edgeless:
  * Not much progress on [package manager lockfile proposal](https://github.com/uapi-group/specifications/issues/70)
  * Talk about reproducible OS image builds with mkosi @ fosdem ([repo](https://github.com/edgelesssys/reproducible-mkosi))
  * uplosi tool for uploading OS images to cloud provider, can pre-calc PCRs for measured boot (started pre-systemd-measure)
* openSUSE MicroOS
  * systemd soft-reboot with transactional-update (switching btrfs root subvolume)
  * Full disk encryption with systemd-pcrlock / signed policies (TPM and FIDO2)
  * Working on BLS support on all architectures
  * Systemd-boot integration in YaST2 Installer and all relevant tooling
* GNOME OS
  * Significant investment in integrating systemd-homed with GNOME (WIP)
  * systemd-sysupdate integration with the GUI app center (GNOME Software)
  * Various improvements to systemd-sysupdate (optional components, bug fixes)
  * Integration of systemd-sysext into GNOME’s CI pipelines \- testing system components as easy as Flatpaks for apps
  * Switch to ukify, drop dracut
  * We now use systemd main branch
* systemd
  * Tons of stuff, see ASG Thursday
* LPC
  * Uapi wishlist room [https://uapi-group.org/kernel-features/](https://uapi-group.org/kernel-features/)
    * contributions welcome (via PR)
  * Session on initrd changes
    * Initrd are quite large with UKIs, need to rm \-rf content, tons of work to do, unnecessary
    * We have some agreement on umounting initrd from the host
    * New root filesystem type as the immutable / (single inode) that can stay around, initrd mounted on top, so it can be removed
    * Page-aligned erofs images packed in uncompressed cpio, kernel could zero-copy use them, so we avoid copying/uncompressing on boot
    * No agreement on immutable via erofs
* mkosi
  * Now fully unprivileged (fs generation via systemd-repart)
    * BTRFS work ongoing to do offline compressing/subvolumes
    * No more bubblewrap/newuidmap
    * mkosi-sandbox instead of bubblewrap
  * More scripts (version, config)
  * Mkosi-initrd as an easy to use tool to build initrds locally
  * Various new distributions
    * Azure Linux
  * OpenSSL engine support for HSM signing
  * (signed) shim, systemd-boot, grub support
  * Sysupdate support
* Red Hat Image Builder / osbuild
  * dm-verity support
    * not yet used in the base image, for automotive use cases
  * bootc-image-builder: [https://github.com/osbuild/bootc-image-builder/](https://github.com/osbuild/bootc-image-builder/)
    * based on rpm-ostree
* [https://uapi-group.org](https://uapi-group.org)
  * FOSDEM CFP open, will prep a proposal Image Based devroom
* Kernel: IPE LSM is now upstream (in v6.12), image-based distros can do code integrity
  * not yet for scripts, there’s ongoing work for that

## Topics

* Dual-booting DPS distros, DPS partition ownership ([https://github.com/uapi-group/specifications/pull/117](https://github.com/uapi-group/specifications/pull/117))
  * New installer for GNOME OS, no longer supports `/etc/fstab`
  * Cannot identify which root/usr partition belongs to which distribution
  * home/srv are shareable
  * two instances of the same os is not supported
  * rootfs discovery is done on pattern matching on labels (implemented in sysupdated, not in the spec yet)
  * corresponding /var partition is found via hashing of the machine-id
    * related: mkosi images with integrated /var currently needs a fixed machine-id
  * Q: Who has adopted DPS in production?
    * Emil: not adopted in SteamOS, as not the full partition set could be discovered
    * We can add label-based matching/filtering. There is an open PR for that.
    * Need backward compat, if there are no labels do the autodiscovery
  * People do want to use F39 and want to try F40 and maybe go back, so multiple versions of the same OS can be useful
  * Pick the rootfs, then the rest is derived (home shared, var hashed by machine-id)
  * Comment: It’s hard to specify `root=` with UKI
    * Solution: new credential, TPM locked
  * `/etc/os-release` add new field, or re-purpose IMAGE\_ID
* Generic, stateless OpenPGP verification of distribution artifacts (repositories, packages, installation media, updates, etc.)
  * Links:
    * [https://github.com/uapi-group/specifications/issues/115](https://github.com/uapi-group/specifications/issues/115)
    * [https://github.com/systemd/mkosi/issues/3042](https://github.com/systemd/mkosi/issues/3042)
    * [https://codeberg.org/heiko/rsop](https://codeberg.org/heiko/rsop)
  * Tons of pitfalls, gpg not standard compliant anymore, is not stateless with regards to keyrings
  * Would be great to have stateless (no keyring) thingy for signatures
  * why not PKCS\#7
  * Apt has global `trusted.gpg.d` directory, being phased out, specified inline in the src line
  * seems similar to how distros handle TLS certificates differently
  * systemd currently could use this for verity authentication  and downloading
  * suggestion: write a UAPI spec (agnostic with regards to OpenPGP or PKCS\#7 or SSH?)
    * PKCS\#7 is x509 so that additional things can be added, like application-specific purpose extensions
    * [https://datatracker.ietf.org/doc/html/rfc5280\#section-4.2.1.12](https://datatracker.ietf.org/doc/html/rfc5280#section-4.2.1.12)
  * Spec says where to put keys, how to name them
  * Key need a purpose, path specifies it, so that a key cannot be used to add trust to another key, only to a repo
  * Should support the usual `/usr/`, `/etc/`, `/run/` priorities
  * Directory structure should be used to find the right key (apt key or sysupdate key), but key policy should be inside the key (signing key or bla key)
    * find key type from the file name suffix
  * Not turn it into a Certificate Store thing, like openssl
  * Have a way to find a private key corresponding to a public key
  * Reserve some OIDs for usage, or define usage extension fields
* would like to add a security feature (BPF LSM) to reject any access to an unauthenticated FS
  * dm-verity/dm-crypt or fixed list (sys, proc, …)
  * also reject any dev nodes outside of /dev
    * chromeos has something similar
  * Reject `AF_UNIX` sockets in `/etc/` or `/usr/`
  * Any other “no-brainer” security policies we should add globally?
  * TODO: file RFE on systemd to track list of policies
  * BPF LSMs are often closed-source currently
  * Make program visible in BPF FS when loaded, so you can do `ConditionPathExists=/sys/filesystem/bpf` (TODO: fix this)
* Think about the relation between FIDO2 \+ TPM2 (tpm2+fido2 as PIN, TPM2 and FIDO2 as recovery key)
  * No two factor auth, it’s either TPM or FIDO
  * Chromeos makes the tpm send a challenge to the fido device, then the TPM reveals the encryption key
    * could likely also be used with PKCS\#7 HSMs
  * SSS (Shamir Secret Sharing) support was being worked on, but keys are combined in software on the CPU, so the spooks can get at you
  * SSH agent could also use this scheme
  * TPM policy expresses that there must be a challenge-answer
  * Q: use this for remote proof of identity?
    * we have PCR15, which includes machine-id and could have other identity identifiers
* immutable without significantly raising the bar for contributors
  * [https://gitlab.com/postmarketOS/postmarketos/-/issues/62\#note\_2095562422](https://gitlab.com/postmarketOS/postmarketos/-/issues/62#note_2095562422)
  * How to deliver an immutable system without raising complexity
  * Need to figure out how to build image
    * Image build efficiency: composefs
  * Q: how about sysexts?
    * Thilo: there are plans to load sysexts very early (initramfs). Also can create systexts on the fly \- using mutable overlays and capturing the writes in a separate “working” directory, then building a new sysext from that
  * perhaps do an overlayfs and keep a list of user-installed packages?
  * This completely breaks the security model
    * For developers, it makes sense for a “break glass” mechanism to do things locally
    * For users, the update mechanism should be designed with security in mind and do transactional updates
    * Think hard on whether the immutable system is the right approach for the use case
  * a way would be to use local keys to build images locally. but uses the full image workflow.
  * They currently use apk and install packages into an overlayfs, purely additive. the overlayfs would become incompatible on update, so nuke it.
  * This kind of issue will become bigger when moving to UKI.
    * Seems like using measured boot is much more interesting than secure boot.
  * Perhaps mkosi approach, keep a list of packages and generate a locally signed sysext. then refresh the sysext
    * Just build the full image locally, no need for sysext if you can sign everything yourself
  * maybe have a “developer mode” like chromeos. there is an entry in the systemd todo list
    * gnome os is looking at this as well
    * have an explicit knob in the TPM for this
    * NIXOS has a similar mode
  * On qualcomm phones there is no way to have a fTPM in TrustZone
* systemd on musl
  * [https://catfox.life/2024/09/05/porting-systemd-to-musl-libc-powered-linux/](https://catfox.life/2024/09/05/porting-systemd-to-musl-libc-powered-linux/)
  * pmOS is for smartphones, they have a high number of contributors relative to the number of users
  * Tons of patches
  * We merged some patches
    * they (adelie folks) use s6 parts to support group shadow stuff that’s required by systemd
  * have a shim library when using systemd on musl
    * `pidfd_spawn`, shadow/gshadow, `printf` string formatter, detected by pkgconfig
    * Maintain shadow parser and printf in parallel with musl
    * needs to be done by libc experts
    * systemd adds more nss stuff (userdb), might be considered advanced use cases
  * locale support in musl is problematic (tests fail)
    * there is conditional execution of tests in systemd already (e.g. when running in a container)
  * Q: how is pmOS handling network
    * A: NetworkManager, due to switching between mobile and wifi
    * GUIs are also deeply integrated with NetworkManager
  * be careful with musl on memory constrained systems, as they don’t release memory back to the OS
* Missing functionality for managing `/etc/` as a writable confext
  * Persisting `/etc/machine-id`
  * Deriving the machine-id for `/var/`, even though `/var/` is probably storing the mutable `/etc/` context
  * Handling presets. Store them in `/usr/` somehow?
  * Possible solution: Just mount `/var/` from initrd? (Who does this actually affect, and do they use image-based / confexts / whatever?)
  * Move mounts that exist on `/etc/` when reapplying, writable `/etc/` on top of the stack
  * Overlayfs works by using the superblock for each elements, pins it, and doesn’t look at the VFS at all, so it needs to be handled by the confext layer
  * How to handle cases where only `/var/` should be persisted?
    * repart uses the ephemeral machine-id to create `/var/`
    * there is now also `systemd.machine-id=firmware`
    * Could add a mechanism to persist machine-id via tpm (NV index, allows factory reset) or system credential
    * could also use the `/var/` partition UUID as machine-id?
      * problematic from the security perspective, needs some authentication
  * perhaps reopen the discussion to mount `/var/` early (in the initramfs)
    * MicroOS already mounts `/var/` (and `/etc/` overlay) in the initrd
    * things are a bit tricky around soft-reboot because there is no initrd
* Support for CC-based (confidential computing) runtime measurements (i.e. "PCRs") in systemd-stub, systemd-pcr\*
  * Most systems have vTPM
  * Should systemd support this if it detects CVM?
    * problematic for Intel, as they do it differently compared to everyone else
      * looks like they will move to a TPM API as well
  * separate PCRs are required for disk encryption via TPM policies, not needed for remote attestation
  * sd stub already supports some CC measurement log
  * Use case: runtime generalised measurements
    * Stuff we build in systemd are not ima based
  * Industry largely is accepting that vTPM is the interface to provide to the OS
    * If userspace uses the 4 intel registers to map to the 4 PCRs systemd uses, this could be hacked to work
  * for very minimal/fixed userspace, tpm might not be needed, as everything can be measured at the start
* Unpriv container support/mountfsd
  * newuidmap is bad
  * We have mountfsd/nsresourced
  * Doesn’t work with containers in the home directory
  * Extend mountfsd to get file descriptor of directory instead of image file
  * Predefine 65k users for containers, no dynamic, no system specific, always the same, statically
  * mountfsd enforces that the setup matches, then checks the parent inode to check that the caller owns it, then set up id mapped mount
  * Dynamic ranges get mapped to this static range
  * Homed already uses id mapping, kernel doesn’t allow nesting. This will be fixed in the kernel to allow multiple mappings.
  * FreeIPA has existing systems with large amounts of mapping, want to make it configurable
    * But if OCI tarballs have to be built offline, no way to synchronise them
  * Mountfsd also needs to learn to delete them, as unpriv users cannot delete files they do not own
  * Once available this may be used for mkosi too
  * IPC is via a varlink unix socket, so they could be used in containers as well via a bind mount
* Partition resizing discussion
  * Should repart start resizing filesystems on its own?
  * There’s nothing to resize vfat. What do we do if we want to grow the ESP?
  * Gnome OS wants to use repart for the installer
  * don’t resize ESP. use xbootloader
  * Android uses dm-linear to runtime-merge partitions
  * DPS GPT flag for extension partition to concatenate them with dm-linear
    * Difficult to implement in repart for the allocation algorithm
  * related: fuchsia has a [blobfs](https://fuchsia.dev/fuchsia-src/concepts/filesystems/blobfs), which makes everything accessible via a rootfs
* Confext dm-crypt+dm-verity loop
  * End goal: being able to reason about every single inode on the system, with different owners/signers per component
  * Agree with this, an issue with composefs because of the multiple layers
* Hermetic `/usr/` \+ Hermetic `/etc/`
  * Easy parts done for MicroOS
  * Some projects can’t have their files split, some refuse patches (some quite rudely)
  * glibc would take patches, but they don’t want to do the work and it needs to be reimplemented in glibc
  * MicroOS currently targets putting everything in `/usr/etc/` and only using that as a fallback if sysadmins don’t put a file in `/etc/` ([libnss\_usrfiles](https://github.com/openSUSE/libnss_usrfiles))
  * tmpfiles \+ `/usr/share/factory/etc/` \+ symlinks seems to work great
  * suggest using volatile /etc and have an allow list for specific stuff
  * for the base os we should be fine now (on fedora), except `/etc/services`
    * MicroOS doesn’t have `/etc/services` any more
    * related: `/etc/protocols`
  * `/etc/shells` is read by many things directly
    * use tmpfiles to create a symlink
  * for glibc, most important is ldconfig, services, nsswitch.conf
    * [https://github.com/systemd/systemd/pull/34535](https://github.com/systemd/systemd/pull/34535)
  * [https://github.com/authselect/authselect/pull/380](https://github.com/authselect/authselect/pull/380)
  * it would be nice if overlayfs had a way to “reveal” (config) files from the lower layers
    * you can diff between upper and lower layers
  * Do we need a arewehermeticyet page? (there’s no “.yet” tld though)
    * Yes
    * Tracker: [https://github.com/uapi-group/specifications/issues/76](https://github.com/uapi-group/specifications/issues/76) (is now a pinned issue)
    * maybe complain about packages which place files in `/etc/`?
    * How to test whether packages?
    * nix patches all packages which write to `/etc/`, can be used as a list of affected packages
    * list of stuff that needs to be in `/etc/` on Arch Linux: [https://github.com/systemd/particleos/blob/main/mkosi.extra/usr/lib/tmpfiles.d/etc.conf](https://github.com/systemd/particleos/blob/main/mkosi.extra/usr/lib/tmpfiles.d/etc.conf)
  * also relates to random top-level paths (mount points in `/`, container usecases, `/opt/`)
* OS Factory Reset
  * What to do with user content on the ESP (credentials, self-signed UKIs)?
  * Original patch set for factory reset became purge, which then tried to delete home directories, which made some people unhappy
    * [https://github.com/systemd/systemd/pull/24983](https://github.com/systemd/systemd/pull/24983)
  * Need to factory reset TPM
    * There is an API for userspace to request this on reboot
  * There should be a way to request factory reset from the boot menu
    * that then triggers repart factory reset, TPM reset and something like bootctl factory-reset
  * Should we have global directories to sort things by owner?
  * Undecidable whether e.g. nVidia drivers should be removed or not, might be required for graphics to work, but maybe the factory reset was initiated to get rid of them
  * let’s add a vendor directory that does not get deleted
* systemd-pcrlock
  * Canonical wrote a whitepaper on PCR-based unlock, but it has not been published yet
  * TODO: ask to get it published
  * SUSE has adopted pcrlock
    * Their older Dell laptops don’t support NV index policies
    * The problem will resolve itself by waiting, until then we need graceful fallbacks
  * Order of measurement is deterministic by PCR, but not globally across all PCRs
  * Golden measurements supported, provided by vendor (in `/usr/`), derive the rest from hardware
    * some laptop vendors provide PCR0 values for LFVS, but the actual measurement values would be better
    * Ideally would supply measurements in the json format for other PCRs touched to the firmware
  * pcrlock is not very user friendly
    * Should have documentation pointing users to the obvious defaults that they should pick
    * `--lock=auto` option like cryptenroll
    * pcrlock status, like bootctl show what local hardware supports
  * pcrlock in combination with signed pcr policies is fixed in v257 via key sharding
* Recovery key fallback
  * Windows doesn’t do much user-friendly stuff, nobody knows their bitlocker recovery key
  * GNOME OS: show GUI when measurements change, to tell the user ‘something’ happened, which PCR is different
  * pcrlock will find a measurement that it can’t make sense of in the log
  * No verb to machine-readable output, can be added
  * This is the remote attestation problem, as you cannot trust the output of a machine booting a compromised kernel
  * There was a bluetooth based project to tie this to a phone (FOSDEM ?? Ultrablue)
    * [https://github.com/ANSSI-FR/ultrablue](https://github.com/ANSSI-FR/ultrablue)
    * User-friendly Lightweight TPM Remote Attestation over Bluetooth [https://archive.fosdem.org/2023/schedule/event/image\_linux\_secureboot\_ultrablue/](https://archive.fosdem.org/2023/schedule/event/image_linux_secureboot_ultrablue/)
* Delivering credentials via systemd-creds to initrd for non-UKI systems (type\#1)?
  * Define UEFI protocol to pass sidecars to sd-stub, which will look for it together with the filesystem-based ones
  * Ostree adds a new menu entry on each system update
  * Btrfs creates a new snapshot on any change to the rootfs (update, config changes, etc)
  * Nixos creates a new entry every time the system config changes
  * Can a credential be used instead of the kernel command line?
    * Problem is GUI: we need a chooser in the menu
    * By opening up the kernel command line choice, we need to ensure we are not opening up a security hole to let a user access the root user

## Action Items

* file RFE on systemd to track list of BPF LSM filesystem lockdown policies
* Make program visible in BPF FS when loaded, so you can do `ConditionPathExists=/sys/filesystem/bpf`
* ask for Canonical whitepaper on PCR-based unlock to be published
* writeup for LWN, send draft to group
* CFP for FOSDEM for image-based devroom
* organise 4th summit next year
  * [CHECK BERLIN MARATHON DATES](https://www.bmw-berlin-marathon.com/en/)
  * Try after ASG, if we can do ASG Tue/Wed
* mastodon posts about systemd v257 features
* add stuff to kernel features wishlist
* file “user friendly pcrlock” issue
* `uki.d` vs `uki.vendor.d` dropins for factory reset
* create PR for certificate directory structure
* make suggestions on other projects to invite (use Signal)
