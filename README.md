# Unyoke your RK3229


### Documentation

- #### Build

     - [Compile OP-TEE, U-Boot and Linux for RK3229](COMPILE.md)

- #### Full Installation

     This approach does not rely on any software that was shipped on your device's onboard flash memory. *This will not work for devices that have onboard NAND flash, as neither mainline U-Boot nor mainline Linux have support for the Rockchip NAND controller.*
     - [U-Boot + Linux Installation (eMMC)](EMMC-INSTALL.md)
     - [U-Boot + Linux Installation (SD/MMC)](SDMMC-INSTALL.md)

- #### Partition Replacement

     This approach uses the bootloader and Rockchip partitioning scheme shipped on your device's onboard flash memory. *This will not work for devices that have onboard NAND flash, as mainline Linux does not have a driver for the Rockchip NAND controller.*
     - [Onboard Flash Partition Replacement](PARTITION-REPLACEMENT.md)


### Testing

- #### Mecer Xtreme Mini S6
    | Procedure                            | Status  | Remarks                       |
    |--------------------------------------|---------|-------------------------------|
    | U-Boot + Linux Installation (eMMC)   | OK      | -                             |
    | U-Boot + Linux Installation (SD/MMC) | OK      | -                             |
    | Onboard Flash Partition Replacement  | OK      | Unresponsive after 30 minutes |


- #### MXQ 4K
    | Procedure                            | Status         | Remarks                                       |
    |--------------------------------------|----------------|-----------------------------------------------|
    | U-Boot + Linux Installation (eMMC)   | Not Applicable | Device has onboard NAND flash                 |
    | U-Boot + Linux Installation (SD/MMC) | OK             | -                                             |
    | Onboard Flash Partition Replacement  | Pending        | No kernel driver for Rockchip NAND controller |

- #### MXQ-Pro 4K
    | Procedure                            | Status         | Remarks                                       |
    |--------------------------------------|----------------|-----------------------------------------------|
    | U-Boot + Linux Installation (eMMC)   | Not Applicable | Device has onboard NAND flash                 |
    | U-Boot + Linux Installation (SD/MMC) | OK             | -                                             |
    | Onboard Flash Partition Replacement  | Pending        | No kernel driver for Rockchip NAND controller |


### Acknowledgement

Thank you to the following people for their assistance in making this process possible.

- Heiko Stübner
- Fabio Bassa
