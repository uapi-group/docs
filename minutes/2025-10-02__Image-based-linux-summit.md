# Image-Based Linux Summit Berlin October 2nd 2025   
🖼️ 🐧 🗻 🥙

# Attendee's projects

* systemd  
* mkosi  
* SUSE: MicroOS/Tumbleweed  
* Red Hat: [image-builder/osbuild](https://osbuild.org), [bootc](https://github.com/containers/bootc), systemd, systemd-boot, composefs  
* Fedora: [Bootc](https://docs.fedoraproject.org/en-US/bootc/getting-started/), [CoreOS](https://fedoraproject.org/coreos/), [Atomic Desktops (Silverblue, Kinoite)](https://fedoraproject.org/atomic-desktops/)  
* Microsoft: confidential containers, [Flatcar](https://www.flatcar.org/), Azure Boost  
* NixOS: systemd-{repart,stub,boot,cryptenroll}, [snix-store](https://snix.dev/about/), [Lanzaboote](https://github.com/nix-community/lanzaboote), UKIs, [NixOS Bootspec](https://github.com/NixOS/rfcs/blob/master/rfcs/0125-bootspec.md), [userborn](https://github.com/nikstur/userborn), [Sécurix](https://github.com/cloud-gouv/securix)  
* Canonical: Ubuntu Core, snaps, foundations/bootloader/package management  
* Freedesktop-sdk  
* GNOME OS  
* Arch Linux: sbctl/go-uefi, signstar(os), mkinitcpio, arch-install-scripts, devtools, buildbtw / vmexec, voa  
* postmarketOS: mkosi and immutable images, dev tooling  
* Devicetree selection for OS installers  
* Cockpit [https://github.com/cockpit-project/cockpit](https://github.com/cockpit-project/cockpit)  
* test.thing [https://codeberg.org/lis/test.thing](https://codeberg.org/lis/test.thing)  
* composefs-rs [https://github.com/containers/composefs-rs](https://github.com/containers/composefs-rs)  
* Aeon Desktop [https://aeondesktop.org](https://aeondesktop.org)  
* tik [https://github.com/sysrich/tik](https://github.com/sysrich/tik)  
* Meta (image builders)  
* 

## Summary of Last Year’s Progress

* Systemd: factory-reset (including TPM), varlinkified lots of building blocks for the installer so GUI progs can call into this IPC to prep disks, UKIs: multi profiles (pcrlock TBD), DTB auto selection, netbooting plus image-based stuff (pxeboot replacement), repart: verity autosizing, verity copyblocks, offline signing, systemd-sbsign for signing EFI and more stuff, with providers support and offline support, fsverity support  
* SUSE: sd-boot for MicroOS, Tumbleweed uses BLS, TPM-based FDE in the YaST2 installer, fido2 \+ tpm2 integration being worked on, sysext support in MicroOS, building MicroOS images with mkosi as root, non-root is WIP, util-linux follows config spec, glib will support it with 2.87.0 too, prototypes to remove SUID from everywhere, new systemd socket activated daemons, one to read and one to modify passwd and shadow entries, simple PAM module making use of it, TIK installer repart based for everything, aeon tpm encrypted by default, fully image-based  
* Composefs: rewrote in rust, can pull OCI images, or local filesystems repos, can use systemd-repart, integrated in bootc project, moving away from ostree, focusing on flow for secure boot signing and bootchain security, UKI and systemd-boot support in bootc  
* Fedora: sysusers.d migration completed for all migrated mostly done in rpm inline as many are based on RPM macros, getting closer to have sd-boot signed, almost finished moving Fedora CoreOS to bootc, focusing on remote attestation, experiment with sysext for Atomic Desktops and CoreOS, not official yet, works well  
* Ubuntu: experimental TPM based FDE for ubuntu classic, ubuntu core IOT related and does it already, many issues with firmwares so verification work, will be ported to classic, to handle common error cases, adding dm-verity for snaps, investigated using DDIs as backend  
  ([https://ubuntu.com/summit](https://ubuntu.com/summit) might be the place to evangalize, Oct 23-24), collaboration with azure for tpm-based CVM, no bootloader, nullboot ([https://github.com/canonical/nullboot](https://github.com/canonical/nullboot)) manages the efi entries directly, uses shim → uki managing efi entries  
* GNOME OS: sd-boot, sysupdate, worked on new installer using repart, pcrlock enabled by default, more testing this summer, issues like fw updates clearing the TPM (not uefi capsule based), problems keeping things unlocked (no recovery key setup sometimes, needs UX improvement), wired up gnome ci to produce sysext images from PRs for easy testing, btrfs installer that does not reboot, puts root btrfs in zram device, encrypt and open luks, then btrfs moves to the disk from the zram ([https://cfp.all-systems-go.io/all-systems-go-2025/talk/QRJVL3/](https://cfp.all-systems-go.io/all-systems-go-2025/talk/QRJVL3/))  
* Arch: worked on stateless verification of artifacts that works with openpgp but extendable (ssh, x509, etc), added to UAPI specs, image based signing enclave using mkosi,   
* NixOS:  
  * dm-verity support in builder (puts Nix Store on /usr/nix/store then bind-mounts to /nix/store). Uses veritysetup generator and verity autosizing.  
  * Removed Perl from mandatory system closure before, [now removed bash](https://github.com/NixOS/nixpkgs/issues/428908).   
  * [Convinced kbd to use .note.dlopen()](https://github.com/legionus/kbd/issues/138).  
    * [Proposed to logrotate](http://github.com/logrotate/logrotate/issues/681) as well but unconvinced we will succeed  
  * Happy about [mount \--beneath support in util-linux](https://github.com/util-linux/util-linux/pull/3680). Used for /etc/ overlay so when system updated it is done atomically  
  * Tried very hard to use the systemd (sysusers and userdbd (no uid allocation) and homed (not declarative, always prompted)) for declarative user management. After 6 months gave up (skill issue?) and developed [Userborn](https://github.com/nikstur/userborn).  
    * Couldn't get homed to be fully declarative  
    * userdbd drop-in files are missing uid allocation and prevention of UID re-use  
  * Lanzaboote: not much progress except on admin/CI work, we are moving to a new git hosting and infra, we hope to resume efforts to be feature parity with stub at that moment. A lot are using it, 2 users claimed having burned their motherboard.  
  * snix-store: a "CA Nix store" for any data, used increasingly, at a slow pace.  
  * Sécurix: a French government-lead project to have a framework to build NixOS hardened workstations with minimal infrastructure, ships with an installer, ships with Lanzaboote, made for teams with an built-in static inventory. Interest to extend to more parties. Semantics not yet image-based.  
    * remaining funny problem: laptops are multi-user, we need to provision disk encryption for all the users who use security keys  
  * systemd on embedded devices:   
    * 16MB of libraries among 1.3MB of libsystemd.so, 2.2MB of libsystemd-core, 4.2MB of libsystemd-shared, 1.1MB of systemd generators, 1MB of udev, 5.4MB of systemd binaries  
  * systemd-pcrlock: we built our own tool to predict pcrlock-like files but we wonder if pcrlock should know how to predict things based on an event log for hardware and a systemd-repart manifest, this works perfectly  
  * machine identity and LoadCredentials: currently, we cannot use socket based LC because reloading rotated credentials is not perfect  
  * Appetite for multi-level security in Spectrum (QubesOS alternative in NixOS): https://git.sr.ht/~alip/syd a competitor to systemd hardening / sandboxing options  
* Flatcar: operationalizing sysexts, building/publishing/hosting: https://extensions.flatcar.org/
  * more APIs/hooks for sysexts to make services merge/unmerge aware  
  * mutable confexts for a migration path from a fully writable to immutable setup, incl. Auto-generating / capturing confexts on the fly   
  * more productization for secure boot  
    *  allow third party keys into sb database so that 3P sysext images can be used


## Topics discussed

* Lennart: Lsm-bpf policies implemented by systemd  
  * How to forbid abstract UNIX sockets in service units  
  * Universal truth: refuse listen on abstract socket  
  * Add new option for refusing abstract socket  
  * Should use VRFs so that service is attached to internet but not interfaces  
  * systemd.resource-control(5) should not contain security options that aren’t about controlling resources  
  * (side note about network-online.target w.r.t. Android semantics: see [https://systemd.io/NETWORK\_ONLINE/](https://systemd.io/NETWORK_ONLINE/) for example of using ping in network-online.taget) or [https://www.freedesktop.org/software/systemd/man/latest/systemd-networkd-wait-online.service.html](https://www.freedesktop.org/software/systemd/man/latest/systemd-networkd-wait-online.service.html)  
  * Add named bpf policies, but some might need to be configurable as individual knobs depending   
    * Use the same scheme as the new OOM policy, unit refers to name, and the policy is configured separately  
    * \`Policy=\` and takes a list of stuff, if that policy needs configuration, we put it in a separate configuration policy file: \`/etc/systemd/policy.d/…\`  
    * Zbigniew: Should be compile-time knob or unit setting instead of a separate new concept  
    * Current policies are parameterized  
    * Should prefer boolean unless necessary, and then it can have a config file  
    * Policies are fine but should be coarse, more higher level than selinux (‘app’ instead of ‘specific binary’)  
  * You need serializing BPF maps and stuff when there’s a vulnerable time window where the system is unprotected, esp. during early boot  
  * There’s an open pull request ([https://github.com/uapi-group/kernel-features/pull/38](https://github.com/uapi-group/kernel-features/pull/38)) to get rid of SUID binaries in the UAPI group. What we can do is we don’t allow SUID execution outside of namespaces via BPF-LSM. We predict that the future of privilege escalation would be to ask nicely to systemd.  
  * Start with the 5 universal truths  
      
    * Universal truth: writable XOR executable mounts (superblock W^X) \[lsmbpf\] (F)  
      * Policy per cgroup (maybe xattr)  
    * Universal truth: device nodes only in /dev/ \[lsmbpf\] (F)  
      * Still allow zero device node (InaccessiblePaths=), null/zero and a few others for container manager  
    * Universal truth: SUID/fcaps are bad \[prctl(), lsmbpf\] (F)  
      * Pam fixed in git main  
      * Polkit fixed in git main  
      * Newidmap/newgidmap need a solution  
    * Universal truth: superblocks must be in-memory, encrypted, authenticated or prohibited (F)  
    * Potential new universal truth: FIFOs and sockets only in /run (incl. XDG\_RUNTIME\_DIR)  
      * putting sockets/fifos in tmpdirs in tests is super super common  
    * Potential new universal truth: fds \[0, 1, 2\] are always open at execve() boundary  
    * Potential new universal truth: nothing mmap’d PROT\_WRITE|PROT\_EXEC (even anonymous?)  
    * Measurement of boot phase ("fuse blowing"), staged boot (F+O)  
    * Separation confext vs. sysext (F)  
    * Use tpm as enclave for fundamental key material (tpm2 in systemd-cryptsetup and systemd-creds) (F+L)  
    * Delegate via sockets (varlink) (L)  
    * Prohibit setuid() to nobody user  
  * Call for new universal policies  
  * Overarching goal of having LSM provided by userspace and out of the kernel  
  * Seccomp filters installed are not done right and this has an impact on performance: the filters are installed individually, issues were opened  
* lis: drafting a Virtual Machine Image API Specification (OEM strings, vsock, serial console, sd\_notify, ssh, ephemeral keys, rootfs resize protocol, etc.) [https://codeberg.org/lis/test.thing/src/branch/main/doc/VirtualMachineAPI.md](https://codeberg.org/lis/test.thing/src/branch/main/doc/VirtualMachineAPI.md)  
  * Shared set of building blocks for distros and harnesses and test frameworks  
  * Ssh is available everywhere already and has tons of features and support (file transfer, “proper session”, port forwarding)  
  * Call it “profile 1” as it will cover 90% but there might be different ones in the future  
  * [https://systemd.io/VM\_INTERFACE/](https://systemd.io/VM_INTERFACE/)   
  * Sd-notify sends systemd-specific string to send ready notification, what about non-systemd guests?  
  * Uapi-group or [systemd.io](http://systemd.io)? Former is more visible, but this has lot of systemd-specific concepts  
  * IMDS needs fully configured network, and then provides json over http for configuring VMs on first boot  
  * Use hwdb for cloud-specific configuration quirks  
  * Profile n. 2 for getting config over a well known port on vsock  
  * Thing in common is credentials as the way to specify configuration, independently from transport protocol  
  * Rename a bunch of credentials to use [uapi-group.org](http://uapi-group.org) instead of [systemd.io](http://systemd.io)   
    and unify .boot. and .stub. Credentials?       
* UEFI firmware and TPM sometimes behave in unexpected ways. It would be nice to have a common benchmark that is Linux focused. Or a database.  
  * Both with ubuntu and gnomeos weird bugs show up with tpm corner cases, especially with certain firmwares  
  * Eg: run fwupd security to check what is missing, pcrlock also can be ran  
  * Document that describes the minimal set of tools to run to verify that everything is green for a new hardware machine, for vendors  
  * Contact Richard Hughes (fwupd) about exposing required information based on hardware?  
  * Add spec to uapi-group, which then can link to various tools  
  * Should include other things like graphics  
  * Talking with Framework, if there was a framework (\!) that was plugin based, it could be a common diagnostic collection solution  
  * First item to check in the test framework: load EFI image with more than 25 sections  
  * Some laptops don’t measure separator even when loading bootloader  
  * Check that firmware updates don’t reset the tpm  
  * Future work: have hwdb tags suggesting hardware support sysexts to load  
  * Build systems for OS images  
  * Flatcar uses custom image build system that creates a disk image w/ partitions, loopback-mounts individual partitions, emerges binaries into the mount point and cleans up portage metadata. Then unmounts and adds verity to the partition.  
  * Bootc-image-builder uses the OSBuild backend to build disk images from bootable containers  
  * Kiwi (used by SUSE) builds all kind of OS and container images  
  * OBS supports mkosi for UKIs and OS images, with secure boot and dm-verity signing  
  * OSBuild creates lvm-ready disk images by using random vg names, as lvm is not namespaced and the build system already runs on lvm  
    * Idea: “mock” lvm and create filesystems images and glue them together with the lvm-specific padding/header  
  * Build systems and sysexts:  
    * Flatcar sysext bakery  
      * Hosts on [github releases](https://flatcar.github.io/sysext-bakery/), with a [URL rewriter](https://github.com/flatcar/sysext-bakery/blob/main/tools/http-url-rewrite-server/Caddyfile)  
      * Nightly GH actions automates building updated sysexts when new upstream versions are released  
    * OBS can build and sign sysexts with mkosi  
    * GNOME OS uses systemd-repart in BuildStream to build and optionally sign sysexts  
      * In the CI sysext-utils is used  
    * Fedora Core OS has a build system on github actions  
* How to split up root vs home partition space  
  * a/b/c vs something else, preallocation problem given limited disk space  
  * Homed: double encryption  
  * If partitioning is used at all, at install time you need to know exactly what the use case will be for the lifetime of the device  
    * Impossible to get right all the time for all users (or arguably impossible to ever get right at any time for any user..but you might get it wrong by a small enough amount to not be noticed)  
  * Adrian: Zero-cost layered encryption ([https://github.com/orgs/uapi-group/discussions/96](https://github.com/orgs/uapi-group/discussions/96))  
  * Have you heard the good news about our lord and saviour composefs?  
    * boo\!   
    * Btrfs\!  
    * You mean ZFS?  
  * Btrfs on sparse volumes  
  * Needs fix in the kernel so that filesystems can know how much space they have available  
  * Btrfs can ask the thin layer for space when it needs it, and can take these volumes in account  
  * Loopback files inside btrfs (or some other writable fs or dm-thin), each loopback file is another instance of btrfs which can be encrypted  
    * Lower level brtfs is the partition table  
    * Could not be btrfs, could be new ‘dm-thin’  
  * Usual blobfs option from fuchsia  
  * Christoph Hellwig proposed object storage, filesystem without files, inodes or everything, just objects that can be allocated, grown, shrunk and written to  
* (NixOS) Ryan: built-in workload/node identity in systemd  
  * [SPIFFE](https://spiffe.io/docs/latest/spiffe-about/overview/): attestation id given to systemd units  
    * Attestation is pluggable: can be done via something like AWS IMDS or TPM2   
    * Local daemon attests that a service is actually that service, and attestation report can then be sent  
    * Service ID should be based on cgroup ID  
    * BPF-LSM to stop random processes to use the cgroup inodes and enforce delegation  
      * Pid1 on boot sets trusted xattr on itself and bpf lsm that stops any other process from setting the same xattr, and enforces delegation rules for cgroups  
  * For generic TPM stuff systemd can take care of it, for more specific use cases implement their own backends via varlink  
* (NixOS) Niklas: How to do Measured Boot with TPM2 encryption with fully image based non-interactive systems? systemd-measure vs systemd-pcrlock, vendor vs. user  
  * (NixOS) Butz: .pcrsig covering more than PCR\#11, e.g. PCR\#4 through a UKI add-on?  
    * PCR11 obvious, PCR4 not  
  * TPM cannot bind policies to static PCRs as it’s too limited, even for simple VMs  
  * Pcrlock solves the consumer/interactive part, observes measurements on first boot, but also can take pcrlock files that can be pre-generated and placed in /usr/  
  * Can provide pcrlock via nul-encrypted creds in the esp (to avoid chicken-and-egg problem)  
  * Updating the policy requires bringing the system in a ‘good state’, which implies rebooting, which is not great for desktops, but might be ok for other cases  
  * Move code from systemd-pcrlock to shared lib so systemd-measure can access it. Should be a simple change and likely easy to get merged.  
  * If the systems are really and completely predictable, pcrsig can be used, and provided via creds too, and it can cover any pcr, not just 11, pick it from the esp via tmpfiles.d  
  * In SUSE sdbootutil to deal with brtfs snapshots and measurements, auto triggered from package manager, rebuilds the policy  
  * Ubuntu Core uses PCR 4, 7, 12 (using custom code)  
  * Related: (NixOS) Butz: automatic/non-interactive migration from PolicyPCR to PolicyAuthorize/PolicyNV for e.g. PCR\#7  
    * You have to re-enroll  
* Sysext services handling on merge/unmerge, service aware sysexts  
  * Users want to merge and unmerge exts at runtime, systemd has no awareness of actual content like tmpfiles or units and viceversa  
  * Flatcar would like services run from sysexts be aware of being run from a sysext so that it can clean up after itself when it is stopped during upgrade/upgraded  
  * Discussion why this is not being done via portable services  
  * Varlink\! Either single or directory multiplexer in sysupdate for callouts on updates  
    * Sysex’d service ships a socket unit that receives a varlink message from sysupdate when it is being unmerged   
* Lennart: Landlock lsm support  
  * Nice because unprivileged, nice design  
  * If it existed 10 years ago it would have been used for ReadOnlyPaths= and friends  
  * Very similar to existing things, but no perfect overlap  
  * [Pending PR on systemd, WIP](https://github.com/systemd/systemd/pull/39174)  
  * Uncertain future, few contributors  
  * Advantage is being able to do this without namespaces  
  * Nix interpreter planning adopting it for **extra** self-sandboxing, e.g. abstract domain sockets (LANDLOCK\_SCOPE\_ABSTRACT\_UNIX\_SOCKET)  
  * Does it fix mount propagation issues with ProtectSystem? Unclear  
  * Pacman uses it for self-sandboxing  
  * Used in GNOME’s localsearch to limit access to what it indexes  
  * And many more individual applications  
* (NixOS) Ryan: systemd-update delta downloads / authentication methods / feedback.  
  * GNOME also needs this  
  * KDE Linux is currently using a stopgap solution inbetween sysupdate [https://invent.kde.org/kde-linux/kde-linux-sysupdated](https://invent.kde.org/kde-linux/kde-linux-sysupdated)  
  * Change importd to use a varlink callout instead of shelling to sd-pull so that it is pluggable  
  * Cutpoints between chunks in casync based on hashing, so best cut point often missed, many file formats have better well-known cut points, like GPT at partition start and end, maybe call out (VARLINK\!) to analysis tool that can give a list of suggested offsets  
  * Documentation of the plan, to-be-implemented by GNOME folks with STF funding:  
    * [https://github.com/systemd/systemd/issues/28227](https://github.com/systemd/systemd/issues/28227)  
    * [https://github.com/systemd/systemd/issues/33351](https://github.com/systemd/systemd/issues/33351)  
    * You could look into using zstd as a patching engine: https://github.com/facebook/zstd/wiki/Zstandard-as-a-patching-engine

# Action Items

* Start a hwdb for firmware where microsoft keys can be safely removed without bricking the machine (or actually, a tri-logic mapping: good, bad, and then the lack of key means “unknown”)  
