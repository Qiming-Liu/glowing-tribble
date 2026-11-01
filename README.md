# cpuid_fault_emulation

## Manual Bazzite kernel releases

Open **Actions ? Manual Bazzite kernel release ? Run workflow** and enter the exact
output of `uname -r` from the target machine. For example:

```text
7.2.9-ogc3.1.fc44.x86_64
```

This workflow runs **only on manual dispatch**. It does not run on pushes, pull
requests, tags, or a schedule. The existing SteamOS workflow is unchanged.

Supported inputs are x86_64 OGC Fedora kernel releases in the format
`x.y.z-ogcR.B.fcN.x86_64`. The exact matching development RPM must still be
available in the official `ghcr.io/opengamingcollective/kernel-packages-fedora`
OCI artifact. The ISO is not needed for compilation, and its kernel may differ
from the installed system after updates.

The workflow fetches only the matching `kernel-devel` RPM, verifies its digest
and package version, and builds in the corresponding Fedora container. It does
not reuse module caches. A missing package, compilation failure, or mismatched
module `vermagic` fails the build and prevents release publication.

Each successful run creates a separate Release tagged
`kernel-<kernel-release>-<run-id>-<run-attempt>`, without replacing the repository's
latest Release. Assets include the `.ko`, `SHA256SUMS`, `BUILD.md`, `vermagic.txt`,
and compiler versions. Download the module and checksum into the same directory:

```bash
sha256sum --check SHA256SUMS
modinfo -F vermagic ./cpuid_fault_emulation-$(uname -r).ko
```

Compilation and matching `vermagic` do not establish runtime compatibility.
The module is unsigned; Secure Boot may require local signing with an enrolled key.
It uses AMD SVM, so a virtual machine must expose SVM/nested virtualization.
No workflow step loads the module or modifies the target machine.
