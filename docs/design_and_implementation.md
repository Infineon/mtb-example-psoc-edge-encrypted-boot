[Click here](../README.md) to view the README.

## Design and implementation

The design of this application is minimalistic to get started with encrypted boot and update of all three project using Edge Protect Bootloader. This code example expects Edge Protect bootloader to be placed in the RRAM. 

All three projects are signed and encrypted (CM33s, CM33ns, CM55) and are placed in placed in external flash. The three projects are encrypted with symmetric AES-128 key(s), while signing this AES key is encrypted by asymmetric EC-P256 public key and added as custom TLV along with encrypted nonce using the ECIES algorithm. The private part of the asymmetric EC-P256 public key is placed at a specific location in the RRAM. For more details, see the Edge Protect Bootloader *README*. Additionally, asymmetric EC-P256(OEM ROT) key private key is used for signing all images, the public part of this key should be provisioned to the device during ownership extension. 

All PSOC&trade; Edge E84 MCU applications have a dual-CPU three-project structure to develop code for the CM33 and CM55 cores. The CM33 core has two separate projects for the secure processing environment (SPE) and non-secure processing environment (NSPE). A project folder consists of various subfolders, each denoting a specific aspect of the project. The three project folders are as follows:

**Table 1. Application projects**

Project | Description
--------|------------------------
*proj_cm33_s* | Project for CM33 secure processing environment (SPE)
*proj_cm33_ns* | Project for CM33 non-secure processing environment (NSPE)
*proj_cm55* | CM55 project

<br>

In this code example, the boot flow begins with the Edge Protect Bootloader, which is launched from the RRAM via the extended boot. The Edge Protect Bootloader is responsible for configuring the QSPI external memory to enable on-the-fly decryption in Execute in Place (XIP) mode.


### Validation and launch

Once the QSPI external memory is configured, the EdgeProtect Bootloader validates all three projects and launches the cm33_s application from the external flash. The cm33_s application is then responsible for launching the cm33_ns application.


### Validation and launch

The cm33_ns application blinks an LED at a fixed interval and subsequently launches the cm55 application from external flash. Notably, all application codes are decrypted on-the-fly by the PSOC&trade; Edge, ensuring secure execution.


### Validation and launch

The Edge Protect Bootloader supports the usage of up to four keys at a time, providing flexibility in key management. This lets you use different symmetric AES-128 keys for each of the three projects along with boot and update images. Additionally, the same key can be used for all three projects if desired.

This code example is configured to use a single AES128 key to encrypt all application images. Edge Proetct Bootloader itself is not encrypted as it is programmed and executed from the internal RRAM Memory.

<br>