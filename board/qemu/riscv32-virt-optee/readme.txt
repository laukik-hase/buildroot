RV32 QEMU virt with OP-TEE in an OpenSBI trusted domain.
Mirror of board/qemu/riscv64-virt-optee from the RISE riscv-optee buildroot
fork, with the domain DT and all components switched to rv32.

Run:

  qemu-system-riscv32 -M virt -cpu rv32,zkr=on -dtb qemu_rv32_virt_domain.dtb -m 2048 -smp 2 -semihosting-config enable=on,target=native -serial mon:stdio -bios u-boot-spl -device loader,file=u-boot.itb,addr=0x80200000 -device virtio-blk-device,drive=hd0 -drive format=raw,file=sdcard.img,id=hd0 -nographic # qemu_riscv32_virt_optee_defconfig

OP-TEE-only smoke test (no U-Boot/Linux; both harts end up idling in the
RPMI REQFWD loop, the untrusted domain traps at 0x81200000 because nothing
is loaded there). Needs qemu-system-riscv64 >= 9.2 (it emulates rv32 via
-cpu rv32), the RISE OpenSBI dev-optee-mpxy-v9 fw_jump.bin built with
PLATFORM_RISCV_XLEN=32, and the DTB from images/ (its source is the DTS added
by linux-patches/, compiled by the kernel build):

  qemu-system-riscv64 -M virt -cpu rv32,zkr=on -m 2048 -smp 2 -nographic -monitor none -serial stdio -no-reboot -dtb qemu_rv32_virt_domain.dtb -bios fw_jump.bin -device loader,file=tee.bin,addr=0xF1000000

OP-TEE must be built with CFG_16550_UART=y to print on the same UART.
Standalone OP-TEE build with the buildroot toolchain (after
"make toolchain" in the buildroot output dir <O>):

  make PLATFORM=virt PLATFORM_FLAVOR=rv32 ARCH=riscv CROSS_COMPILE32=<O>/host/bin/riscv32-buildroot-linux-gnu- CFG_16550_UART=y CFG_RISCV_PLIC=n CFG_TDDRAM_START=0xF1000000 CFG_TDDRAM_SIZE=0x01000000
