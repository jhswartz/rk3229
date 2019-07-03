# Unyoke your RK3229


### Documentation

- #### Build

     - [Compile OP-TEE, U-Boot and Linux for RK3229](COMPILE.md)

- #### Full Installation

     This approach does not rely on any software that was shipped on your device's onboard flash memory. *Installation to onboard NAND flash is currently not supported in the current U-Boot configuration.*
     - [U-Boot + Linux Installation (eMMC)](EMMC-INSTALL.md)
     - [U-Boot + Linux Installation (SD/MMC)](SDMMC-INSTALL.md)

- #### Partition Replacement

     This approach uses the bootloader and Rockchip partitioning scheme shipped on your device's onboard flash memory. *This is the only internal approach that is likely to work with the current U-Boot configuration for boards that feature onboard NAND flash instead of an eMMC.*
     - [Onboard Flash Partition Replacement](PARTITION-REPLACEMENT.md)


### Testing

- #### Mecer Xtreme Mini S6
    | Procedure                            | Status  | Notes                         |
    |--------------------------------------|---------|-------------------------------|
    | U-Boot + Linux Installation (eMMC)   | OK      | -                             |
    | U-Boot + Linux Installation (SD/MMC) | OK      | -                             |
    | Onboard Flash Partition Replacement  | OK      | Unresponsive after 30 minutes |


- #### MXQ 4K
    | Procedure                            | Status         | Notes                         |
    |--------------------------------------|----------------|-------------------------------|
    | U-Boot + Linux Installation (eMMC)   | Not Applicable | Device has onboard NAND flash |
    | U-Boot + Linux Installation (SD/MMC) | OK             | -                             |
    | Onboard Flash Partition Replacement  | Pending        | -                             |

- #### MXQ-Pro 4K
    | Procedure                            | Status         | Notes                         |
    |--------------------------------------|----------------|-------------------------------|
    | U-Boot + Linux Installation (eMMC)   | Not Applicable | Device has onboard NAND flash |
    | U-Boot + Linux Installation (SD/MMC) | OK             | -                             |
    | Onboard Flash Partition Replacement  | Pending        | -                             |


### Acknowledgement

Thank you to the following people for their assistance in making this process possible.

- Heiko Stübner
- Fabio Bassa
