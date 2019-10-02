This was tested with MCUBoot v1.3.1 and Zephyr v2.0.0!

To enable MCUBoot (and mcumgr) to have the secondary slot stored in the SPI flash the following config changes have to applied:

For MCUBoot:

```
diff --git a/boot/zephyr/prj.conf b/boot/zephyr/prj.conf
index dd6733353..7d3bb09de 100644
--- a/boot/zephyr/prj.conf
+++ b/boot/zephyr/prj.conf
@@ -36,7 +36,7 @@ CONFIG_BOOT_SIGNATURE_KEY_FILE="root-rsa-2048.pem"
 
 CONFIG_FLASH=y
 
-CONFIG_MULTITHREADING=n
+CONFIG_MULTITHREADING=y
 
 ### Various Zephyr boards enable features that we don't want.
 # CONFIG_BT is not set
@@ -47,3 +47,13 @@ CONFIG_LOG=y
 CONFIG_LOG_IMMEDIATE=y
 ### Ensure Zephyr logging changes don't use more resources
 CONFIG_LOG_DEFAULT_LEVEL=0
+
+CONFIG_SPI=y
+CONFIG_SPI_2=y
+CONFIG_SPI_NRFX=y
+CONFIG_FLASH=y
+CONFIG_SPI_NOR=y
+CONFIG_SPI_NOR_FLASH_LAYOUT_PAGE_SIZE=4096
+CONFIG_SPI_NOR_PAGE_SIZE=256
+CONFIG_SPI_NOR_SECTOR_SIZE=4096
+CONFIG_BOOT_MAX_IMG_SECTORS=256
```

For Zephyr:

```
diff --git a/boards/arm/nrf52840_pca10056/nrf52840_pca10056.dts b/boards/arm/nrf52840_pca10056/nrf52840_pca10056.dts
index d90b1dd684..32c9d7711d 100644
--- a/boards/arm/nrf52840_pca10056/nrf52840_pca10056.dts
+++ b/boards/arm/nrf52840_pca10056/nrf52840_pca10056.dts
@@ -176,7 +176,7 @@ arduino_i2c: &i2c0 {
 	mosi-pin = <20>;
 	miso-pin = <21>;
 	cs-gpios = <&gpio0 17 0>, <&gpio1 5 0>;
-	mx25r64: mx25r6435f@0 {
+	flash1: mx25r6435f@0 {
 		compatible = "jedec,spi-nor";
 		reg = <0>;
 		spi-max-frequency = <80000000>;
@@ -184,6 +184,8 @@ arduino_i2c: &i2c0 {
 		jedec-id = [c2 28 17];
 		size = <67108864>;
 		has-be32k;
+		erase-block-size = <4096>;
+		write-block-size = <4>;
 		wp-gpios = <&gpio0 22 0>;
 		hold-gpios = <&gpio0 23 0>;
 	};
@@ -212,11 +214,7 @@ arduino_spi: &spi3 {
 		};
 		slot0_partition: partition@c000 {
 			label = "image-0";
-			reg = <0x0000C000 0x000067000>;
-		};
-		slot1_partition: partition@73000 {
-			label = "image-1";
-			reg = <0x00073000 0x000067000>;
+			reg = <0x0000C000 0x0000ce000>;
 		};
 		scratch_partition: partition@da000 {
 			label = "image-scratch";
@@ -236,6 +234,19 @@ arduino_spi: &spi3 {
 	};
 };
 
+&flash1 {
+	partitions {
+		compatible = "fixed-partitions";
+		#address-cells = <1>;
+		#size-cells = <1>;
+
+		slot1_partition: partition@0 {
+			label = "image-1";
+			reg = <0x00000000 0x0000ce000>;
+		};
+	};
+};
+
 &usbd {
 	compatible = "nordic,nrf-usbd";
 	status = "okay";
diff --git a/samples/subsys/mgmt/mcumgr/smp_svr/prj.conf b/samples/subsys/mgmt/mcumgr/smp_svr/prj.conf
index baaadc04af..4aeb948b3b 100644
--- a/samples/subsys/mgmt/mcumgr/smp_svr/prj.conf
+++ b/samples/subsys/mgmt/mcumgr/smp_svr/prj.conf
@@ -37,6 +37,14 @@ CONFIG_MCUMGR_CMD_OS_MGMT=y
 CONFIG_MCUMGR_CMD_STAT_MGMT=y
 
 ### nRF5 specific settings
+CONFIG_SPI=y
+CONFIG_SPI_2=y
+CONFIG_SPI_NRFX=y
+CONFIG_FLASH=y
+CONFIG_SPI_NOR=y
+CONFIG_SPI_NOR_FLASH_LAYOUT_PAGE_SIZE=4096
+CONFIG_SPI_NOR_PAGE_SIZE=256
+CONFIG_SPI_NOR_SECTOR_SIZE=4096
 
 # Specify the location of the NFFS file system.
 CONFIG_FS_NFFS_FLASH_DEV_NAME="NRF_FLASH_DRV_NAME"
```