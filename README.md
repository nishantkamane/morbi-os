Morbi OS — Immutable Fedora-based OCI Image for Edge Systems
=============================================================

[![Releases](https://img.shields.io/badge/Releases-Download-blue?style=for-the-badge&logo=github)](https://github.com/nishantkamane/morbi-os/releases) [![Build Status](https://github.com/nishantkamane/morbi-os/actions/workflows/build.yml/badge.svg)](https://github.com/nishantkamane/morbi-os/actions/workflows/build.yml)

🐧 🔒 📦

Overview
--------

Morbi OS is an image-based, immutable Linux distribution built on Fedora OSTree and packaged as an OCI image. It targets edge and appliance use cases that need a stable, auditable base system and container-friendly deployment. The image uses OSTree for system delivery and rpm-ostree for client-side management. The build pipeline integrates BlueBuild and GitHub Actions to produce signed and unsigned images.

Feature status: experimental.

Key features
------------

- Immutable root filesystem via OSTree.
- Image delivery as OCI/registry artifacts.
- Signed image support for verified boots and updates.
- Atomic upgrades and rollback with rpm-ostree.
- Minimal base for containers and edge workloads.
- CI with BlueBuild and GitHub Actions to produce reproducible builds.

Badges and links
----------------

- Releases: [https://github.com/nishantkamane/morbi-os/releases](https://github.com/nishantkamane/morbi-os/releases)  
  Download the release asset and run the installer as described in the Releases section below.

- Build badge: links to the repository workflow and shows the latest build status.

Quick image preview
-------------------

![Fedora OSTree image](https://upload.wikimedia.org/wikipedia/commons/3/3f/Fedora_logo.png)  
A minimal OSTree image that works with rpm-ostree and OCI registries.

Topics
------

atomic, bluebuild, bluebuild-image, custom-image, image-based, immutable, linux, linux-custom-image, oci, oci-image, operating-system

Getting started — client install
-------------------------------

This section covers how to rebase an existing rpm-ostree system to Morbi OS images. Use these commands on a Fedora Atomic or rpm-ostree host.

1. Rebase to the unsigned image first. This installs necessary keys and policies for signed images.
   ```
   sudo rpm-ostree rebase ostree-unverified-registry:ghcr.io/nishantkamane/morbi-os:latest
   ```
2. Reboot to apply the rebase.
   ```
   sudo systemctl reboot
   ```
3. After reboot, rebase to the signed image for verification and secure updates.
   ```
   sudo rpm-ostree rebase ostree-image-signed:docker://ghcr.io/nishantkamane/morbi-os:latest
   ```
4. Reboot again to switch to the signed image.
   ```
   sudo systemctl reboot
   ```

Commands assume the image tags are published to GitHub Container Registry (ghcr.io). Replace the repository host and image tag as needed.

Image variants and tags
-----------------------

Morbi OS publishes the following artifacts:

- ostree-unverified-registry:ghcr.io/nishantkamane/morbi-os:latest — unsigned test image.
- ostree-image-signed:docker://ghcr.io/nishantkamane/morbi-os:latest — signed image suitable for production.
- OCI tarball and installer assets in Releases.

Use the tags that match your environment and trust policy.

Install from Releases
---------------------

Visit the release page and download the installer asset. The release page is here:

https://github.com/nishantkamane/morbi-os/releases

On the release page look for an installer file named morbi-os-installer.sh or a tarball that contains the OSTree commit and metadata. Download the installer and run it on a host that supports rpm-ostree.

Example steps:

1. Download the installer (replace tag or asset name as needed).
   ```
   curl -L -o morbi-os-installer.sh "https://github.com/nishantkamane/morbi-os/releases/download/v1.0.0/morbi-os-installer.sh"
   ```
2. Make it executable and run it.
   ```
   chmod +x morbi-os-installer.sh
   sudo ./morbi-os-installer.sh
   ```
The installer configures the local host to use the Morbi OSTree commit. It may perform a rebase and schedule a reboot.

If the asset name differs, download the correct file from the Releases page and follow the asset readme.

BlueBuild and CI
----------------

The repository uses BlueBuild as the declarative build engine. The pipeline does the following:

- Build a minimal Fedora base tree.
- Install configured packages.
- Commit the tree to OSTree.
- Produce OCI images and tarballs.
- Sign images and generate verification metadata.
- Publish artifacts to registry and to the Releases page.

Use the BlueBuild docs for setup and to create your own template:

https://blue-build.org/how-to/setup/

CI steps (high level):

- GitHub Actions runs BlueBuild in a container.
- If the pipeline passes, it packages artifacts and adds them to the release.
- Releases host signed and unsigned images.

Customize the pipeline by editing the BlueBuild YAML in the repo. You can add packages, adjust systemd units, and change the image layout.

Building locally
----------------

You can build Morbi OS locally with BlueBuild and podman/docker. Typical steps:

1. Install BlueBuild and container runtime.
2. Clone the repository.
3. Run BlueBuild against the build manifest.
   ```
   bluebuild build bluebuild.yaml
   ```
4. Inspect the generated OSTree commit and OCI image in the artifacts folder.

Packaging and signing
---------------------

Signing helps you enforce verified boot and trusted updates. The pipeline produces:

- ostree repo files with GPG signatures.
- OCI image signatures using cosign or similar tooling.
- Metadata files that rpm-ostree can use for verification.

If you host your own signing keys, configure the BlueBuild manifest to use your keys. The build process will inject key material into the signing steps and produce a signed image.

Runtime and containers
----------------------

Morbi OS works well in container-driven deployments. Use the OCI image as a base for container workloads or deploy it to devices that support container runtimes.

Running an image locally with podman:
```
podman pull ghcr.io/nishantkamane/morbi-os:latest
podman run --rm -it ghcr.io/nishantkamane/morbi-os:latest /bin/bash
```
That container exposes a minimal OS runtime. Add systemd and other packages in your build manifest for full system behavior.

Updating and rollback
---------------------

- rpm-ostree update will fetch the next commit and prepare it as a layered change.
- Reboot to apply the new deployment.
- To rollback, use:
  ```
  sudo rpm-ostree rollback
  sudo systemctl reboot
  ```

Debug and troubleshooting
-------------------------

- Check OSTree status:
  ```
  rpm-ostree status
  ostree admin status
  ```
- Check logs:
  ```
  journalctl -b -u rpm-ostreed
  journalctl -xe
  ```
- Verify image signatures with cosign or the tool used by the pipeline.
- If rebase fails, verify registry access and image tags.

Releases and assets
-------------------

The official release page contains installer scripts, signed tarballs, and SHA256 checksums. Visit the releases page, download the installer asset, and run it to install Morbi OS on a test host.

https://github.com/nishantkamane/morbi-os/releases

Look for these common assets in a release:

- morbi-os-installer.sh — automated installer script.
- morbi-os-ostree.tar.xz — OSTree repo tarball.
- morbi-os-oci.tar.gz — OCI image tarball.
- checksums.txt — SHA256 checksums for artifacts.

Security and signing
--------------------

- The pipeline supports OSTree GPG signing for commits.
- OCI images can sign with cosign.
- Configure verification policies on clients to trust your keys.
- If you use unsigned images for test, switch to signed images before production.

Contributing
------------

- Fork the repository.
- Create a feature branch.
- Add or modify the BlueBuild manifest to add packages or change the layout.
- Test builds locally.
- Open a pull request with your changes and a short description of the goal.

Design decisions
----------------

- OSTree gives atomic upgrades and rollbacks. It suits devices that need durable, tested images.
- OCI delivery matches common registry workflows and integrates with containers and Kubernetes.
- BlueBuild favors declarative builds. The manifest holds the system definition and packages.

Repository topics and metadata
------------------------------

Topics included in the project metadata:
- atomic
- bluebuild
- bluebuild-image
- custom-image
- image-based
- immutable
- linux
- linux-custom-image
- oci
- oci-image
- operating-system

License
-------

Check the LICENSE file in the repository for the full terms. The base content uses Fedora packages licensed under their upstream licenses. Check each package for its license.

Acknowledgements
----------------

- Fedora and rpm-ostree projects for upstream tooling.
- BlueBuild for the build engine and pipeline model.
- GitHub Actions for CI integration.

Files and layout to check in the repository
------------------------------------------

- bluebuild.yaml — build manifest.
- cloud-init/ or ignition/ — example firstboot config.
- images/ — output images and test data.
- install/ — installer assets and helper scripts.
- .github/workflows/build.yml — CI pipeline.

Contact and support
-------------------

Open issues on GitHub to report bugs or request features. Include logs and reproduction steps when possible.

Releases (again)
----------------

Visit the release page to download the installer and artifacts, then run the installer file from the release assets:

https://github.com/nishantkamane/morbi-os/releases

Follow the asset instructions for your chosen release and tag.