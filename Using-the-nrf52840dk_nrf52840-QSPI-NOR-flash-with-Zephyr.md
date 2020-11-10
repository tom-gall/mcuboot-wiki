To enable MCUBoot (and mcumgr) to use the secondary slot stored in the QSPI NOR flash someone can use the provided overlay and configuration files under `boot/zephyr/boards`.

To build MCUboot:

```
west build -b nrf52840dk_nrf52840 -d build_boot bootloader/mcuboot/boot/zephyr -- -DDTC_OVERLAY_FILE=$(pwd)/bootloader/mcuboot/boot/zephyr/boards/nrf52840dk_qspi_nor_secondary.overlay -DOVERLAY_CONFIG="$(pwd)/bootloader/mcuboot/boot/zephyr/boards/nrf52840dk_qspi_nor.conf;$(pwd)/bootloader/mcuboot/boot/zephyr/boards/nrf52840dk_qspi_secondary_boot.conf
west flash -d build_boot
```

To build an app (smp_svr in the example):

```
west build -b nrf52840dk_nrf52840 zephyr/samples/subsys/mgmt/mcumgr/smp_svr -- -DDTC_OVERLAY_FILE=$(pwd)/bootloader/mcuboot/boot/zephyr/boards/nrf52840dk_qspi_nor_secondary.overlay -DOVERLAY_CONFIG="overlay-bt.conf;$(pwd)/bootloader/mcuboot/boot/zephyr/boards/nrf52840dk_qspi_nor.conf"
west sign -t imgtool -- --key bootloader/mcuboot/root-rsa-2048.pem -v 1.0.0
west flash --hex-file build/zephyr/zephyr.signed.hex
```