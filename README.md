shim transparent loader demo
============================

This is a simple proof of concept illustrating the use of shim (with
the transparent loader enhancements) to load iPXE, which in turn boots
an operating system.

Quick start
-----------

 1. git submodule update --init

 2. make

 3. Append "secboot.db" to the machine's Secure Boot "db" database

 4. Configure machine to perform a UEFI PXE boot

 5. Expose this directory via TFTP and HTTP

 6. Configure DHCP to boot shim.efi followed by menu.ipxe:

        if exists user-class and option user-class = "iPXE" {
          filename "http://this.web.server/shimdemo/menu.ipxe";
        } else {
          next-server this.tftp.server;
          filename "shimdemo/shim.efi";
        }

Components
----------

There are two submodules: one for shim (with the transparent loader
enhancements) and one for iPXE.

There are two self-signed certificates: "secboot" and "vendor".

There is one external certificate: "fedora" (extracted from a signed
Fedora kernel).

Only the "secboot" certificate is added to the machine's "db" database
(i.e. the Secure Boot certificate whitelist).

The "shim.efi" binary is built to trust both the "vendor" and "fedora"
certificates, and is itself signed with the "secboot" certificate.

The "ipxe.efi" binary is signed with the "vendor" certificate.

Demo
----

The machine performs a UEFI PXE network boot, and is instructed to
download "shim.efi".  Since "shim.efi" has been signed with the
"secboot" certificate, and the "secboot" certificate is present in the
machine's "db" database, "shim.efi" is permitted to execute.

"shim.efi" then locates and boots "ipxe.efi".  Since "ipxe.efi" has
been signed with the "vendor" certificate, and "shim.efi" trusts the
"vendor" certificate, "ipxe.efi" is permitted to execute.

"ipxe.efi" displays the boot menu and then boots the selected
operating system.  The operating system may be signed with either the
"secboot", "vendor", or "fedora" certificates.

This therefore allows a boot sequence of:

 1. UEFI PXE network boot

 2. shim.efi (signed by "secboot" certificate, enrolled in "db")

 3. ipxe.efi (signed by "vendor" certificate)

 4. Fedora kernel (signed by "fedora" certificate)
