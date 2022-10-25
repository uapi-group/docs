# Image-based Linux summit, 5th - 6th October 2022, Berlin

The first image-based Linux summit took place on the 5th and the 6th of October, 2022. It was conducted as a loosely structured BoF-style event.
The following parties were present at the summit (in no particular order):
- Distros / Entities: Ubuntu Core, Debian, GNOME OS, Fedora CoreOS, Endless OS, Arch Linux, SuSE, Flatcar, Microsoft, Amazon, Meta
- Projects: systemd, image-builder/osbuild, mkosi, tpm2-software, System Transparency, buildstream, BTRFS, rpm-ostree

## Actionables and tasks
Action items listed here can be found as **TODO #[x]** in the full meeting minutes below, with [x] being the action item number.

1. Provide feedback to the [proposal for `systemd-syscfg`](https://github.com/systemd/systemd/issues/24864)
2. Propose spec for application configuration file search path order to evangelise to application builders. 
   Extend xdg_base specification to specify order (first to last):
    - `/run` , then `/etc` , then `/usr/etc`
    - use `/usr/share/factory` for vendor defaults, apps should never look there
    - Let users override the above, e.g. via env vars (e.g. add `/usr/local/…`)
3. Extend UKI to support allow-list of kernel command line parameters, or env var-like file
    - allow sd-stub to pick up new cmdline via files, make shim verify it (via MoK), and consume it
4. Add support for UKI booting to Grub. E.g. Grub module that loads a UKI and jumps into the kernel.
    - Grub already has PE parser
    - **4a. TODO sd-boot**: Support type 1 entry that references type 2 + own kernel command line options
5. Missing mkosi features:
    - make sysext, UKI builds fully reproducible. calculate the signature server side, rebuild locally and apply (to make sig optional in case of legacy / non-UEFI system)
    - support iso9660 - [tracked here](https://github.com/systemd/mkosi/issues/1220), also see **TODO 10**
    - switch to systemd-repart for unprivileged image builds, [tracked here](https://github.com/systemd/mkosi/issues/1228)
    - SPDX manifest file **TODO 17**
6. Extend Discoverable Partitions Specification to
    - add policy language to restrict what the initrd will load from the disk
    - allow recovery partition GUID
    - allow discovery subvolumes/inode (BTRFS)
7. Add versioning of resources to sysext / UKI spec. Use RPM versioning format. Auto-detect versioned resources by means of version prefix in sub-directories:
    - `./foo.a/` has image(s) for resource *foo* version *a* (with *a* following RPM versioning format)
    - `./foo.b/` has image(s) for resource *foo* version *b*
8. (Flatcar team) Add support for `boot-complete.target` to signal "healthy boot". Flatcar uses non-standard `update_engine.service` for the same.
    - Tracked in [Flatcar #865](https://github.com/flatcar/Flatcar/issues/865)
9. Systemd `boot-complete.target`: add optional timer that fails if target is not reached within timeout, triggering a reboot, in turn triggering a rollback.
10. Support in `systemd-repart` for
    - optionally removing outdated partitions to free up space.
    - MBR
    - iso9660 (will also address part of **TODO 5**)
11. Systemd to provide / standardise on a common target that means, after this is reached, root fs has been resized/reparted/etc and it is fully writable and ready
    - `initrd-root-fs.target` already exists for the initrd stage
    - `local-fs-pre.target` ? or new target?
    - https://github.com/systemd/systemd/pull/24680 
12. Standardise UEFI environment for systemd-repart to trigger full / partial reset.
    - Also define a systemd target to reboot into for factory reset (full or partial) for interactive / desktop systems to offer on a boot menu
    - Add support to repart for full repartitioning (full factory reset) if the disk UUID was set to `0xFFFF`… (similar to existing support for partition factory reset on UUID `00000`…)
    - To handle app config, add support to systemd-tempfiles for removing files in factory reset mode - [tracked here](https://github.com/systemd/systemd/pull/24983)
13. Discoverable Partition Spec: add specifications for partition flags to indicate the partition's purpose (e.g. boot, sysext, recovery, etc.)
    - systemd-sysupdate would ignore recovery partitions and not consider these part of an A/B update scheme 
14. systemd-sysupdate, systemd boot: add protection against unwanted roll-back (downgrade attack) by use of TPM counters.
15. Add support for secure boot SBAT csv of images embedded in UKI, add SBAT csv to verity signature in DDI discover part
    - mkinitcpio already implements objcopy functions which could be reused
16. systemd-sysupdate download helper to become ‘pluggable’, so casync or other can be used instead of the built-in downloader (http).
17. Add SPDX/SBOM field in os-release that points to local file or URL **TODO #17**
18. Enhance either cryptsetup or systemd-cryptsetup to support Shamir Secret Sharing and combine multiple tokens/slots

## Meeting minutes

### OS images and integrity
- UKI (unified kernel image): [a new kernel/initrd format](https://github.com/systemd/systemd/pull/24351)
   - limited to 4GB to fit into EFI space
   - Secure, immutable, all in a single file (easy to update)
   - Baked-in initrd, cmdline, root credentials (signed list of TPM PCR 11 values, key that signs the list)
   - simplified storage of OS secrets, like disk encryption keys, making updates less brittle
   - Credentials stored in the ESP, together with the UKI, are also gathered and passed to the booted system
   - wrapped in a PE file, signed for UEFI Secure Boot
   - There is a Red Hat feature request to allow multiple kernel cmdline in a single UKI, bootloader generates menu based on that (eg: debug mode, safe mode, etc.)
   - Lack of bootloader-passed, custom command line could create issues later. **TODO #3** to fix this.
- Root fs: UKI cmdline can have verity roothash, credentials can extended that
- UKIs enable both secure boot and TPM based security, independently
- The TPM-based story for UKIs gives stronger guarantees than just Secure Boot, by allowing to express/attach policies to specific kernes/UKIs.
- grub lacks support for UKI. **TODO #4** to fix that.
    - would limit future flexibility in UKIs though (e.g. custom logic) since Grub would then need to support that, too
- Initrd is generated via mkosi
   - https://github.com/systemd/mkosi-initrd
   - [LPC presentation slides](https://lpc.events/event/16/contributions/1188/attachments/993/1949/lpc2022-new-design-for-initrds.pdf)
      - [LPC presentation recording](https://www.youtube.com/watch?v=rDPo5onf-Rc&t=11088s)
   - Layered initrd, signed/read-only/measured, built server-side and deployed and loaded on demand
   - Base initrd in UKI works on most systems, is extendable via sysext (e.g. optional support for hardware)
   - mkosi-initrd can build base and layers from packages only
   - Some work is ongoing on dracut to support this, in a hybrid mode where it builds the initrd/extensions from dracut modules, but no dracut runtime logic on the running system
      - Problem: size constraints for existing systems, that have to be supported

- Documentation on pre-calculated and signed PCR values:
   - [Unrendered measure manpage, updated quickly](https://github.com/systemd/systemd/blob/main/man/systemd-measure.xml)
   - [Rendered measure manpage, may be outdated](https://www.freedesktop.org/software/systemd/man/systemd-measure.html)
   - [Unrendered stub manpage, updated quickly](https://github.com/systemd/systemd/blob/main/man/systemd-stub.xml)
   - [Rendered stub manpage, may be outdated](https://www.freedesktop.org/software/systemd/man/systemd-stub.html)

- UKI might be proposed for Fedora 39
   - Sysext initrd extensions will be shipped in RPMs
   - Sysext, UKI not fully reproducible, see **TODO #5**

- [Discoverable Partitions Specification](https://systemd.io/DISCOVERABLE_PARTITIONS/)
   - Please file PRs to extend the specification to [systemd's repository](https://github.com/systemd/systemd/blob/main/docs/DISCOVERABLE_PARTITIONS.md)
   - Input collected in Summit see **TODO #6**
   - DDI: Discoverable Disk Image, uses DPS and signed dm-verity. Inspired by Canonical's Snaps, uses the same technique and builds on top of it.

- Distro-independent sysext images? Vision: Kubernetes project can publish their own Kubernetes sysext, runs well on all distros, is self-updating client side, so Kubernetes upstream can keep it up to date just by publishing new versions.
   - Could be done by promoting building tools for Go/Rust static binaries. Flatcar has a lab / playground for this, [for example for Docker](https://github.com/flatcar/sysext-bakery).

- `mkosi` to build UKIs, DDIs etc.. python based
   - configured via .ini plus drop-ins
   - supports dm-verity, SecureBoot, Signatures, PCR measurement
- for building UKIs, DDIs, sysext, initrd, OS images (w/ split partition support for individual partition update images)
   - Produces manifest file with list of packages that were installed (SBOM)
   - Does not support SLSA
   - Native support on OBS [Open Build Service](https://build.opensuse.org/project/show/home:bluca:mkosi)
   - Some features missing like reproducible builds, iso9660, switching to systemd-repart, see **TODO 5**
      - WIP: unprivileged builds, but dracut still needs root (run nspawn in image), BTRFS too
      - Fedora CoreOS uses a small VM to handle partitioning / fill partitions w/o privileges, circumventing the above
- rpm-ostree + coreos-assembler can build images (kind-of-images as you can put them into a container image / convert to something else), see https://github.com/coreos/rpm-ostree, https://github.com/coreos/coreos-assembler
- [osbuild](https://www.osbuild.org/)
- Endless: OSTree built with eos-ostree-builder - private repo, but [deb-ostree-builder](https://github.com/dbnicholson/deb-ostree-builder) is similar – and GPT disk images with [eos-image-builder](https://github.com/endlessm/eos-image-builder)
- KIWI https://github.com/OSInside/kiwi 
- Flatcar uses custom python script inherited from CoreOS, always builds images from sources (Gentoo style)
   - Adds [SLSA provenance](https://www.flatcar.org/docs/latest/reference/supply-chain/) (sources fingerprints -> binaries fingerprints) to OS image during build.

- Need to standardize on a common target that means, after this is reached, root fs has been resized/reparted/etc and it is fully writable and ready - see **TODO #11**

- Novel Linux file system technology for immutable OS images (blobfs, …)
   - [composefs](https://github.com/containers/composefs)
   - RPM-backed images, e.g. using btrfs reflinks (rpm2extents) or FUSE (repomount)
   - Blobfs ([Fuchsia](https://fuchsia.dev/fuchsia-src/concepts/filesystems/blobfs))
      - simple FS that only allows git-like object store. Add object, get hash back
      - No Linux support
      - No inodes/filenames/dentries/attributes/access modes/timestamps/directories
      - verity-protected, de-duplicated by default, garbage collection
      - Simple enough to be accessed from firmware too
- Would enable use of single partition instead of A/B, could be added to the bottom of GPT and grow at the front. Pool of images - DDIs/UKIs or anything else - available everywhere with implicit and automatic trust in that partition.

### Handling user configuration changes
- systemd already supports `/usr/share/factory/` and specifiers in tmpfiles.d to populate `/etc/` from there
- Other projects (ostree, libeconf) populate from `/usr/etc/`
- Some other projects use a 3-way merge of configurations in `/etc/`
   1. Previous version default config
   2. Previous version w/ user changes
   3. New version default config
- Merging of configurations? PAM as a negative example w/ single-line changes
   - Red Hat works around this by abstracting PAM into features (coarse application support), Debian has a more fine-grained (but manual) merge tool.
   - PAM future: serve a static configuration and provide an API for dynamic / user defined changes
- PAM is just one example, glibc's nss-switch is another

- OSTree copies from `/usr/etc/` to `/etc/` at first boot
   - On update, new configs are copied, user changes remain untouched (== config merge in per-file granularity)
- OpenSuSE uses the [libeconf](https://opensuse.github.io/libeconf/) project, and MicroOS uses BTRFS snapshots: when a new snapshot is created, chroot into it and run the rpm scriptlets, then makes it read-only. User modifications are done via OverlayFS and stored in /var, which is outside the snapshot.
- Programs could forward-convert config via ExecStartPre that figures out config differences semantically
   - sshd has ssh-keygen for example
   - Not fully adopted on all distros for first boot generation 
   - needs buy-in of each project to ship such tool
- User management: Fedora has started moving away from `/etc/passwd` and `/etc/group` to `systemd-sysusers`

- systemd project just published [syscfg proposal](https://github.com/systemd/systemd/issues/24864), see **TODO #1**
   - Core concept is that even for configuration, one can always trace any file back to its source in a cryptographically secure way
   - Could systemd-syscfg look at configuration-release `SYSCFG_LEVEL=` and make sure it matches `/usr/`'s `SYSCFG_LEVEL=`, and find the right image, or refuse a wrong one?
   - Add sliding window for version support, on top of `==``
   - systemd-repart will be able to build syscfg DDIs, should be multi-call (call as e.g. systemd-makesyscfg build config DDI)
- long term, systemd sysext aims to use `/etc/` as immutable / service factory default stack of configuration images and `/etc/local/` as local user modifications.
   - distinguish between verified, secure config and binaries
   - make node state cryptographically deterministic and only deploy workloads if we can prove a node to run trusted binaries in a trusted configuration
   - tie a specific configuration to a specific time window, which allows e.g. a secret to be unlocked only in a given week and not afterwards

- syscfg rollback: how to do it at system level?
   - At service level (`ConfigurationImages=`) it’s easier as there’s a single one that has a clear signal on reload
   - At system level problem is combinatorial explosion, could try with counters - each config image has a counter, start with highest and count down
   - Also needs rollback protection, idea: TPM2 monotonic counter establishes (sliding) range window of allowed versions

- **TODO #2**: establish spec for search path order, to at least try to push applications, extend xdg_base specification to specify order as (first to last). See details there.
  - `/run/` -> `/etc/` -> `/usr/etc/`, `/usr/share/factory/` for vendor
  - mask, override, etc.
- OSTree uses `/usr/etc/` and `/etc/`, prefers local (`/etc/`) files in case of conflicts
   - enforces reboots after config changes
- Some distros address this at packaging level, push package maintainers to default to `/usr/etc/`
   - [UsrEtc on SuSE](https://en.opensuse.org/openSUSE:Packaging_UsrEtc)

- credential handling in `/etc/`: `systemd-credentials` can inject sensitive information into system services
  - secrets are encrypted using TPMs, and only decrypted on demand immediately before consumption
  - meant as a replacement for passing creds in env variables
  - exposes secrets as files

### Updating OS Images and overlays
#### Prior Art
- Build/deploy: can we learn from OCI?
   - Current OCI users use tarballs and merge them, no offline/online security
   - OCI layers are linear and one way merge
- Fedora IoT / RHEL 4 Edge uses [Greenboot](https://github.com/fedora-iot/greenboot)
   - During meeting Greenboot was nudged to use boot-complete
- SuSE has openSUSE [Health-checker](https://github.com/openSUSE/health-checker)
- Ubuntu has a health check too
- GNOME OS [nudge to use boot-complete](https://gitlab.gnome.org/GNOME/gnome-build-meta/-/issues/560)
- Endless OS has rudimentary check, "did a user GNOME session start? If so, grub-editenv - unset recordfail")

#### Future Developments
- systemd-repart: declarative configuration of system partitions, can populate systems with files too, cryptographically secure
    - runs in initrd, "magic tar for creating system layout"
    - can build images, too (DDI - Discoverable Disk Image)
    - Supports GPT
    - Some features missing, see **TODO #10**

- Automatic boot assessment after update to mark partition/disk/whatever as good/bad
   - [Specification](https://systemd.io/AUTOMATIC_BOOT_ASSESSMENT/)
   - Systemd provides `boot-complete.target` to signal "boot is healthy"
      - **TODO #8**: Add support to Flatcar
   - Already hooked up with sd-boot
   - Make this target depend on health test / critical service units
      - There is one test shipped by upstream that checks for no failed services, but is likely too broad as some services should be allowed to fail
   - **TODO #9**: add optional timer that fails if nothing happens within time, triggering reboot / rollback


### General A/B booting, versioning
#### Prior Art
-  grub vs. sd-boot
   - legacy hardware / non-EFI
   - access to filesystems in grub
   - shim only supports grub (signature) as bootloader for secure boot currently
      - Issue could be mitigated if grub could boot UKIs
- Using partitions vs. putting fs images into writable space
  - OS (initrd) would need to trust FS used for writable space
  - but since space is writable, FS exploits can be staged there, tainting the FS
  - OS, when booting, mounts tainted FS to get access to OS FS images stored there, and chain of trust is broken
- Chromebook/Android
- Work in progress for (rpm-)ostree based systems to [establish trust in the partition content](https://github.com/containers/composefs)
- State of the "custom/partial" [BLS support in Fedora](https://fedoraproject.org/wiki/Changes/BootLoaderSpecByDefault)
- Support for fallback in Shim/grub
   - [Pull request](https://github.com/rhboot/shim/pull/502)
   - Used mostly for updating shim itself
- Non-EFI platforms?
   - See **TODO #4** Grub UKI support
- [SteamOS EFI loader](https://gitlab.com/evlaV/steamos-efi)
   - Uses a "A" or "B" partition labels
- [RAUC](https://github.com/rauc/rauc)
   - Embedded updater, with A/B slots, integrates with many bootloaders/filesystems

#### Future Developments
- Concept: flag partition after booting, good/bad, fallback
- sd-boot adds counters to UKI filename, uses counters for ordering in the menu for default entry
- More generally, any resource (sysext or UKI) could be versioned with two counters
   - e.g.: `nspawn --image=/path/some/directory/` picks the image from that directory based on counters
   - Version format is modified RPM version format
   - Directory suffix to signal that a directory organizes objects with this version specification inside. e.g.: 
      - `./foo.a/` has image(s) for resource *foo* version *a* (with *a* following RPM versioning format)
      - `./foo.b/` has image(s) for resource *foo* version *b*
   - **TODO #7**: add to sysext spec
- **TODO #10**: repart remove old / outdated partition that is not useful anymore
   - Problem: usr/root partition is fixed after initial boot and repartition

### TPM provisioning, enrolling, measurement, remote attestation
- Set up a Linux PCR registry, with a public list of which project claims which PCR, to allow vendor-independent coordination
- [Keylime](https://keylime.dev/) is used in SUSE
   - Measures using IMA hashes
   - Initrd measured twice
- Fedora CoreOS also looking into keylime for on boot remote attestation
- DICE as a reference architecture
   - [TCG specification](https://trustedcomputinggroup.org/wp-content/uploads/TCG_DICE_Attestation_Architecture_r22_02dec2020.pdf)
   - [Documentation from Microsoft](https://www.microsoft.com/en-us/research/project/dice-device-identifier-composition-engine/)

### OS factory reset
#### Prior Art
- Fedora CoreOS has a [WIP tracking issue to support factory reset](https://github.com/coreos/fedora-coreos-tracker/issues/399)
- Endless OS currently just erases users & data, implemented as enabling a [systemd service for next boot which disables itself once run](https://github.com/endlessm/eos-boot-helper/tree/master/factory-reset)
- Flatcar uses a flag file in the ESP which is picked up and acted on by Ignition in the initrd, causing re-deployment of user config
- Ubuntu core supports [reset via recovery partition (image file in ESP)](https://ubuntu.com/core/docs/use-recovery-mode)
- SuSE MicroOS keeps BTRS snapshot #1 around, and it can be selected from cli or bootloader menu

#### Future Developments
- Full reset vs. selectively keeping wanted local data
   - Selective reset would allow reconfiguration for OSes which apply config at provisioning time (Flatcar, FCOS).
   - large downloads (e.g.container images, databases) would remain while config could be updated
   - benefit over using configuration management (Chef, Ansible) is that deployments remain idempotent, no config drift
   - **TODO #12** has a sub-item to add support for removing files to systemd-tmpfiles to better support partial reset
- systemd-repart can mark a partition for factory reset, will re-initialise / clear those parts first, then continue boot
   - Reset is triggered by an UEFI variable - in repart, but other systems use different ways (see below). **TODO #12** tracks standardising on reset.
      - Also align on high-level target to reboot into, so desktops can provide menu to request factory reset
      - Repart also supports filling the partition if the UUID is set to 0000…
      - Add the same for 0xFFFFF to signal full factory reset (full repartitioning) for the disk GPT
- How to restore factory config, and apps stored outside the OS image?
   - Tmpfiles support for removing stuff only on factory reset mode - [tracked here](https://github.com/systemd/systemd/pull/24983)
- Handling recovery partitions (Ubuntu core)
   - Image could be updated but [usually isn’t (security vs. robustness)](https://ubuntu.com/tutorials/how-to-ubuntu-core-recovery-mode#1-overview)
   - **TODO #13** update Discoverable Partition Spec to define flags indicating purpose (boot, sysext, recovery). sysupdate would not consider recovery partitions in A/B update scheme.

### Distributing OS images
#### Prior Art
- rpm-ostree builds OS images on the server side and distributes the resulting tree
   - Optionally, a new layer can be built  locally and overlaid on the base
   - When the base image is updated then the local build + overlay are repeated
   - Multiple transport channels are supported, e.g. static website (with static deltas, or individual object pulls, similar to Git). container registry
   - Initial installation uses image from (Live) ISO (w/ installer for automation), but image can also be `dd`ed to disk directly
   - 3 release streams, user chooses which stream to follow

- Endless OS
   - initial install is from GPT disk image `dd`ed to disk
   - updates are ostree images, composed server-side from debs. No local layering (like rpm-ostree does)
   - Transport channel is static HTTP
   - Clients fetch a delta from current revision, if available on server; otherwise, missing files are pulled individually

- Ubuntu core distributes a filesystem image that can be `dd`ed to disk
   - Everything is shipped as snaps (in squashfs images)
      - on boot, writable space contains 1 or more versions of base OS squashfs, "current" / "good" one is mounted on /.
      - Apps are snaps too, mounted similarly.
   - Xdelta for image (snaps) updates
   - Transport channel is http with proprietary server side
      - supports detached signatures based on hashes of snaps
      - Xdelta deals with squashfs mostly fine, deltas are created on-demand server-side

- Flatcar: initial provisioning is full image, via cloud store / marketplace or manual download from image server (https). Image contains full partition layout; root partition is extended during provisioning to fill whole disk
   - Flatcar updates are full images, no delta. Combined kernel+initrd+OS partition, all signed
   - Update from any version to any other.
   - Updates are orchestrated using the Omaha protocol, (Chrome OS update protocol)
      - Omaha is a "control plane" layer on top of transports; clients poll for update
      - Allows controlled roll-out (only X clients at the same time)
      - Feedback channel from client to server, so server "knows" how many clients succeeded / failed. Roll-out can be stopped if too many clients fail
      - "update succeeded" uses "boot succeeded" semantics, i.e. user-defined system critical service units must start successfully, else update failed
      - Implemented in [Nebraska server](https://github.com/kinvolk/nebraska), is FOSS
   - Update transport channel is HTTPS, but Nebraska is transport agnostic and could serve any URIs
   - Flatcar offers 4 distinct channels: Alpha (developer), Beta (production-ready, for canaries) and Stable. LTS is a "golden stable" that receives only patch updates for 18 months.
      - Patch releases go directly to each channel, new major releases are stabilised via Alpha -> Beta -> Stable transition

#### Future Developments
- Systemd-sysupdate
   - Is an updater for DDIs, but could  also do directory trees. Command line tool to apply updates.
   - Information on how to update is inside the images themselves
   - Point sysupdate to the image, and it will be updated if needed (current version lower than image)
   - Built around lists of objects, version compared
   - Manifest of object store server-side as sha512 sums file
   - Compares version of what’s local, and what’s on the server, and gets right thing
   - Will be able to use casync as transport, which chunks an image (like zsync) into files (unlike zsync). Hence [zsync](http://zsync.moria.org.uk/) needs http range requests server-side, [casync](https://github.com/systemd/casync) does not.
   - Configurable chunk/block size; download size vs granularity trade-off left to owner/user
   - Giant pile of files (== individual chunks) on server, to be CDN-friendly
   - Always reconstruct target from seed, ideally could be used for backups (e.g.: homed)
   - Idea: create a TPM signed report which includes manifest for images, and also metadata about node itself (PSI etc), orchestrator collects these from nodes on a timer and produces credential that can only be decrypted by that specific node in that specific state
   - sysupdate transport is built-in (http) but should be pluggable - see **TODO #16**

- TPM counters could be used to guard against unwanted rollbacks (downgrade attack).
   - To still allow regular rollback,sliding window of allowed counter values could be used
   - Counters can only go up, need root to change. TPM can be reset by root, but lose all keys. Updating moves window forward.
   - Systemd at boot will bump counter, with graceful fallback in case counters do not work/not available
   - Vendors increase upper end on new release, increase lower end when cutting out older release
   - Signed PCR policy will include info about acceptable counters window
   - Client will be brought up to speed, as counter can be increased by any value
   - TPM monotonic counter support in tpm2-tools: https://www.mankier.com/1/tpm2_nvincrement
   - By spec, at least 3 counters per TPM, but nothing uses them anywhere
   - Does not allow multi-boot systems, as counters would not match
   - Tracked in **TODO #14** systemd to support rollback protection with counters
- Shim supports [Secure Boot Advanced Targeting (SBAT)](https://github.com/rhboot/shim/blob/main/SBAT.md), stores counter in signed EFI variable, supports multiple vendors/distros.
   - When booting a new version, old version can be marked as invalid, as EFI variables encode component names and minimum version
   - uses authenticated variables to store the policy (distro + version)
   - allows multi-boot systems, and cross-vendor/downstream distributions
   - sd-boot/sd-stub already embeds SBAT csv
   - initrd could be covered, too, with additional optional SBAT entries
   - **TODO #15** add support for SBAT of inner UKI images (initrd), add SBAT to verity signature JSON in DDI, if kernel includes SBAT either sd-boot needs to check it through shim, or combined them in the outer UKI SBAT

- SBOM
   - mkosi builds and publishes manifest file in custom JSON
   - Flatcar has an SPDX SBOM, a JSON file with version/license info, now it also added [SLSA provenance](https://flatcar-linux.org/docs/latest/reference/supply-chain/)
   - [SuSE](https://documentation.suse.com/sbp/server-linux/html/SBP-SLSA4/index.html)
   - [SPDX](https://spdx.dev/) and CyclonDX to build a standard manifest file
   - provenance/tracking
     - [uswid](https://github.com/hughsie/python-uswid)
     - [coswid](https://datatracker.ietf.org/doc/draft-ietf-sacm-coswid/)
     - [slsa](https://slsa.dev/provenance/v0.1)
   - **TODO #17**: add SPDX support to mkosi, [tracked here](https://github.com/systemd/mkosi/issues/1230)
   - **TODO #17**: add SPDX field to extension-release/os-release
SPDX=<URL> or SPDX=<path> for self-contained DDI

- Live updates for kernel and userspace w/o downtime
   - "rocket surgery", many pitfalls
   - driver states / network queues etc. a problem, needs 100% control over hardware
      - some HW state like TPM measurements are not resettable
   - kexec can handle kernel w/o downtime, and w/o impacting user space
   - CRIU to update userland? Start userland in a VM and then move userland processes using CRIU
   - But what state CRIU migrates is opt-in, all things to carry over need to be specified explicitly
   - systemd-suspend uses freezer cgroup to stop everything except pid1
      - But needs to re-arrange cgroup tree, which might break things
   - systemd interested in hierarchical model of freezing, in which states can be passed up
     - extend File Descriptor store so that state is persisted across kexec too (opt-in), using persistent memory, so that for a service a restart of the service or a kexec are the same
   - userspace "reboot" (possibly with an "exitrd", which is different from "initrd", and resets some kernel state like `/dev/`), shut down everything and re-initialise to `default.target`, after switching to new root/usr DDI
     - But need to figure out how to deal with TPM secrets that are phase-specific, but not different from kexec
     - Could stop measuring after the "first" boot phases

