# QEMU aarch64 emulation

The following script should allow booting the aarch64-linux UEFI
installer ISOs provided through the `nixos:unstable-small` and
`nixos:trunk` jobsets, when no native hardware support exists.

Pass the ISO file as the first argument, and everything should boot up.

```bash
#!/usr/bin/env nix-shell
#!nix-shell -i bash -p qemu_full virt-viewer
# shellcheck shell=bash

set -euo pipefail

BASEDIR=$(dirname "$0")

VMNAME=nixos-aarch64
ISO="${BASEDIR}/$1"
NVRAM="${BASEDIR}/vars-pflash.raw"
CORES=6
MEMORY=4096
KEYBOARD="de"

cleanup() {
	kill "$QEMU_PID" || true
}

trap cleanup EXIT

run_vm() {
	OVMF_FD=$(nix-build '<nixpkgs>' --no-out-link -A OVMF.fd --system aarch64-linux)
	test -f "${NVRAM}" || {
		cp "${OVMF_FD}/AAVMF/vars-template-pflash.raw" "${NVRAM}"
		chmod u+w "${NVRAM}"
	}

	qemu-system-aarch64 \
		-name $VMNAME,process=$VMNAME \
		-machine virt,gic-version=max \
		--accel tcg,thread=multi \
		-cpu max \
		-smp $CORES \
		-m $MEMORY \
		-k $KEYBOARD \
		-serial stdio \
		-drive if=pflash,format=raw,file="${OVMF_FD}/AAVMF//QEMU_EFI-pflash.raw",readonly=on \
		-drive if=pflash,format=raw,file="${NVRAM}" \
		-device virtio-scsi-pci \
		-device virtio-gpu-pci \
		-device virtio-net-pci,netdev=wan \
		-netdev user,id=wan \
		-device virtio-rng-pci,rng=rng0 \
		-object rng-random,filename=/dev/urandom,id=rng0 \
		-device virtio-serial-pci \
		-drive file="${ISO}",media=cdrom \
		-boot d

	QEMU_PID=$!
}

run_vm
````
