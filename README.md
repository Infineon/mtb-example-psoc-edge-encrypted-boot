# PSOC&trade; Edge MCU: Encrypted boot

This code example demonstrates image encryption, execution, and updates using EdgeProtect bootloader. It showcases a secure workflow for protecting external memory using encryption capabilities of Infineon's PSOC&trade; Edge MCU.

This code example has a three project structure: CM33 secure, CM33 non-secure, and CM55 projects. All three projects are programmed to the external QSPI flash and executed in Execute in Place (XIP) mode. Extended boot launches the CM33 secure project from a fixed location in the external flash, which then configures the protection settings and launches the CM33 non-secure application. Additionally, CM33 non-secure application enables CM55 CPU and launches the CM55 application.

[View this README on GitHub.](https://github.com/Infineon/mtb-example-psoc-edge-encrypted-boot)

[Provide feedback on this code example.](https://cypress.co1.qualtrics.com/jfe/form/SV_1NTns53sK2yiljn?Q_EED=eyJVbmlxdWUgRG9jIElkIjoiQ0UyNDE1MjEiLCJTcGVjIE51bWJlciI6IjAwMi00MTUyMSIsIkRvYyBUaXRsZSI6IlBTT0MmdHJhZGU7IEVkZ2UgTUNVOiBFbmNyeXB0ZWQgYm9vdCIsInJpZCI6InJhdmlraXJhbiIsIkRvYyB2ZXJzaW9uIjoiMi4wLjEiLCJEb2MgTGFuZ3VhZ2UiOiJFbmdsaXNoIiwiRG9jIERpdmlzaW9uIjoiTUNEIiwiRG9jIEJVIjoiSUNXIiwiRG9jIEZhbWlseSI6IlBTT0MifQ==)

See the [Design and implementation](docs/design_and_implementation.md) for the functional description of this code example.


## Requirements

- [ModusToolbox&trade;](https://www.infineon.com/modustoolbox) v3.6 or later (tested with v3.6)
- Board support package (BSP) minimum required version for:
   - KIT_PSE84_EVAL_EPC2: v1.0.0
   - KIT_PSE84_EVAL_EPC4: v1.0.0
- Programming language: C
- Associated parts: All [PSOC&trade; Edge E84 MCU](https://www.infineon.com/products/microcontroller/32-bit-psoc-arm-cortex/32-bit-psoc-edge-arm/psoc-edge-e84) parts


## Supported toolchains (make variable 'TOOLCHAIN')

- GNU Arm&reg; Embedded Compiler v14.2.1 (`GCC_ARM`) – Default value of `TOOLCHAIN`
- Arm&reg; Compiler v6.22 (`ARM`)
- IAR C/C++ Compiler v9.50.2 (`IAR`)
- LLVM Embedded Toolchain for Arm&reg; v19.1.5 (`LLVM_ARM`)


## Supported kits (make variable 'TARGET')

- [PSOC&trade; Edge E84 Evaluation Kit](https://www.infineon.com/KIT_PSE84_EVAL) (`KIT_PSE84_EVAL_EPC2`) – Default value of `TARGET`
- [PSOC&trade; Edge E84 Evaluation Kit](https://www.infineon.com/KIT_PSE84_EVAL) (`KIT_PSE84_EVAL_EPC4`)


## Hardware setup

See the kit user guide to ensure that the board is configured correctly.

Ensure the following jumper and pin configuration on board.
- BOOT SW must be in the HIGH/ON position
- J20 and J21 must be in the tristate/not connected (NC) position


## Software setup

See the [ModusToolbox&trade; tools package installation guide](https://www.infineon.com/ModusToolboxInstallguide) for information about installing and configuring the tools package.

Install a terminal emulator if you do not have one. Instructions in this document use [Tera Term](https://teratermproject.github.io/index-en.html).

This example requires no additional software or tools.


## Operation

To ensure the encryption functions correctly, note that this code example has certain prerequisites that must be met, and it is essential to follow the instructions outlined below:

1. Enable secure boot flow
2. Program the key encryption key (KEK)
3. Enable postbuild signing for this example
4. Build, program, and run this example


### Enable secure boot flow 


#### Prerequisite

Infineon’s Edge Protect Tools provides a set of command line tools used to perform functions required for key signing, key generation, OEM certificate creation, device provisioning, and so on. These tools are executed through a Shell tool. **Edge Protect Tools** executable is available in the **Edge Protect Security Suite v1.6**, located in the *C:/Users/\<username>/Infineon/Tools/ModusToolbox-Edge-Protect-Security-Suite-1.6/tools/edgeprotecttools/bin/* directory.

Add the executable path to the system environment path variable of the host PC.

To use Edge Protect Tools CLI, is recommended to use "modus-shell", which is installed along with ModusToolbox&trade; located in the *ModusToolbox/tools_x.y* directory. 


#### Ownership transfer

Transfer the ownership of the device to yourself before changing the policy file. Follow the steps to transfer ownership. Note that steps 1 to 9 are optional if you have already enabled secured boot on your device – You can then jump on to Step 10 directly.

1. Open modus-shell and navigate to the work directory where you want to initialize Edge Protect tools and created associated assets such as keys and policy

    ```
    cd <app-directory>

    ```

2. Execute the following command to initialize the tools. This is required once the new version of EAP is installed
   
   - For EPC2 device, use this command:
    
        ```
        edgeprotecttools -t pse8xs2 init
        ```
    
    - For EPC4 device, use this command:
        ```
        edgeprotecttools -t pse8xs4 init
        ```

3. Execute the following command to configure the openOCD tools path:

    ```
    edgeprotecttools set-ocd --name openocd --path <openocd_path>
    ```

    > **Note:** Replace <openocd_path> with the path to the openocd directory . Typically, this will be C:/Infineon/Tools/ModusToolboxProgtools-1.5/openocd.

4. Create a private and public key pair. The following command generates one pair of keys that is placed in the keys directory:

    ```
    edgeprotecttools create-key --key-type ECDSA-P256 --output keys/oem_private_key_0.pem keys/oem_public_key_0.pem 
    ```

5. To generate a new CSR, execute this command:

    ```
    edgeprotecttools -t pse8xs2 oem-csr --certificate-name "oem-cert" --oem "Dummy OEM" --project "Dummy Project" --project-number "1234" --public-key-0 keys/oem_public_key_0.pem --cert-type development --output packets/apps/prov_oem/oem_csr.bin --sign-key-0 keys/oem_private_key_0.pem
    ```

6. Submit the generated CSR to **Edge Protect Signing Service** [here](https://osts.infineon.com/) to generate the Infineon-signed OEM certificate and download the generated certificate

   **Figure 1. Submit CSR to generate signed certificate**
    
   ![](images/epss-csr-submission.jpg)

7. Provision the device with the new key and certificate to transfer ownserhip

    ```
    edgeprotecttools -t pse8xs2 provision-device -p policy/policy_oem_provisioning.json --key keys/oem_private_key_0.pem --ifx-oem-cert packets/apps/prov_oem/oem_cert.bin
    ```

    > **Note:** See [AN237849](https://www.infineon.com/AN237849) for more details on transfer of ownership


#### Enable secure boot in extended boot

To enable secure boot in the PSOC&trade; Edge device, provision it with the `secure_boot` flag set to 'true' in the OEM policy. 

The OEM policy file (*policy_oem_provisioning.json*) is located in the *[application directory]/policy/* directory, which is created when `edgeprotecttools` is initialized. For `edgeprotecttools` initilization, see Section 2.2.2.1 of "Getting started with PSOC&trade; Edge security".

8. In the OEM policy, set the `extended_boot_policy` > `secure_boot` > `value` to 'true': 

    ```
      "extended_boot_policy": {
        "secure_boot": {
          "description": "Disable/Enable secure boot option",
          "value": true
        }
    ```

9. Once the policy is updated, provision the device with the updated policy

    ```
    edgeprotecttools -t pse8xs2 provision-device -p policy/policy_oem_provisioning.json --key keys/oem_private_key_0.pem
    ```

    For provisioning details, see Section 2.2.2.4 of [AN237849](https://www.infineon.com/AN237849)


### Generate and program/provision encryption KEK

10. Genearte the AES-128 Key for image encryption using this command:

    ```
    edgeprotecttools create-key --key-type AES128 --output keys/aes128_key.bin
    ```

11. Generate the key encryption key (KEK) using this command:
 
    ```
    edgeprotecttools create-key --key-type ECDSA-P256 --output keys/enc-ec256-priv.pem keys/enc-ec256-pub.pem
    ```
 
12. Convert the pem private key to the DER-PKCS8 format using this command:
 
    ```
    edgeprotecttools convert-key -k keys/enc-ec256-priv.pem -o keys/enc-ec256-priv.bin -f DER-PKCS8
    ```

13. Program the encryption keys:
 
    Program private key to the device using openocd at the start address of the "kek_region" defined in the memory map. Check your RRAM memory map in *Device Configurator* for correct location of the **kek_region**. Default configuration places **kek_region** in *0x32039000*.

    - For EPC2 device, use this command:

        ```
        openocd.exe -s scripts -f interface/kitprog3.cfg -f target/infineon/pse84xgxs2.cfg -c "init; reset init; flash write_image keys/enc-ec256-priv.bin 0x32039000; reset; shutdown"
        ```
            
    - For EPC4 device, use this command:

        ```
        openocd.exe -s scripts -f interface/kitprog3.cfg -f target/infineon/pse84xgxs24.cfg -c "init; reset init; flash write_image keys/enc-ec256-priv.bin 0x32039000; reset; shutdown"
        ```


### Enable postbuild signing for this example

Once the device is succesfully provisioned to enable the secure boot feature, extended boot will launch the first user application only if the image signature has been succesfully verified. 

14. To boot the application successfully, sign the first user application (*proj_cm33_s*) with the same key you used for taking the device ownership  

15. Open the *common.mk* file located in the top-level directory of your application and update the `COMBINE_SIGN_JSON` as shown:

    ```
    COMBINE_SIGN_JSON?=./bsps/TARGET_$(TARGET)/config/GeneratedSource/boot_with_bldr.json
    ```
 
### Add bootloader to PSOC_Edge_Encrypted_Boot

16. Add proj_bootloader to this code example by follow the steps #2 to #7 in the Operation section of the [**Edge Protect Bootloader**](https://github.com/Infineon/mtb-example-edge-protect-bootloader) code example's *README.md*.

> **Note:** 
> 1. Edge Protect Bootloader README.md describes how to add the bootloader to Basic Secure App. Add it to this code example (**PSOC_Edge_Encrypted_Boot**) instead
> 2. **(For the LLVM_ARM compiler):**  Edge Protect Bootloader does not support LLVM_ARM. In the proj_bootloader directory, edit the *Makefile* to force other supported compilers for EPB (for example: TOOLCHAIN=GCC_ARM)


### Generate debug launch configurations from combiner signer JSON file

Combiner Signer JSON file must contain the `extra_config` option to generate debug launch configuration for the signed hex file. All combiner signer JSON files in this code example already contains the required configuration to generate ModusToolbox&trade; launch configurations.

17. Whenever the combiner signer file used in the *common.mk* file is changed, navigate to the application-directory in a terminal window and perform the following step: 

    <details><summary><b>In Eclipse IDE</b></summary> 

      make eclipse

    </details>
    
    <details><summary><b>In other IDEs</b></summary>

      Follow the instructions in your preferred IDE

    </details> <br>
 
    Alternatively, click on the Generate launch config button in the **Quick Launch** panel to genearte the launch configurations


### Build, program, and run the examples

See [Using the code example](docs/using_the_code_example.md) for instructions on creating a project, opening it in various supported IDEs, and performing tasks such as building, programming, and debugging the application within the respective IDEs.

18. Connect the board to your PC using the provided USB cable through the KitProg3 USB connector

19. Open a terminal program and select the KitProg3 COM port. Set the serial port parameters to 8N1 and 115200 baud

20. After programming, the application starts automatically. Confirm that "PSOC Edge MCU: Encrypted Boot" is displayed on the UART terminal

    **Figure 2. Terminal output on program startup**

   ![](images/terminal-enc-boot.png)
   

21. Confirm that the kit user LED1 blinks at approximately 1 Hz

### Performing updates

To check how to update the encrypted image, follow these instructions: 

22. Open the *common.mk* file located in the top-level directory of your application and update the `COMBINE_SIGN_JSON` as shown:

    ```
    COMBINE_SIGN_JSON?=./bsps/TARGET_$(TARGET)/config/GeneratedSource/boot_with_bldr_upgr.json
    ```

23. In the same *common.mk* file, set `IMG_TYPE` to update: 
    
    ```
    IMG_TYPE?=UPDATE
    ```

24. Build and program the image built for Update. See [Using the code example](docs/using_the_code_example.md) for instructions on building, programming the application in various supported IDEs. 

25. After programming, the Edge Protect Bootloader performs the update, as shown here: 

    **Figure 3. Terminal output on application update**
    
    ![](images/terminal-enc-update.png)

## Related resources

Resources  | Links
-----------|----------------------------------
Application notes  | [AN235935](https://www.infineon.com/AN235935) – Getting started with PSOC&trade; Edge E84 MCU on ModusToolbox&trade; software
Code examples  | [Using ModusToolbox&trade;](https://github.com/Infineon/Code-Examples-for-ModusToolbox-Software) on GitHub
Device documentation | [PSOC&trade; Edge E84 MCU datasheet](https://www.infineon.com/products/microcontroller/32-bit-psoc-arm-cortex/32-bit-psoc-edge-arm#documents) <br> [PSOC&trade; Edge E84 MCU reference manuals](https://www.infineon.com/products/microcontroller/32-bit-psoc-arm-cortex/32-bit-psoc-edge-arm#documents)
Development kits | Select your kits from the [Evaluation board finder](https://www.infineon.com/cms/en/design-support/finder-selection-tools/product-finder/evaluation-board)
Libraries  | [mtb-dsl-pse8xxgp](https://github.com/Infineon/mtb-dsl-pse8xxgp) – Device support library for PSE8XXGP <br> [retarget-io](https://github.com/Infineon/retarget-io) – Utility library to retarget STDIO messages to a UART port
Tools  | [ModusToolbox&trade;](https://www.infineon.com/modustoolbox) – ModusToolbox&trade; software is a collection of easy-to-use libraries and tools enabling rapid development with Infineon MCUs for applications ranging from wireless and cloud-connected systems, edge AI/ML, embedded sense and control, to wired USB connectivity using PSOC&trade; Industrial/IoT MCUs, AIROC&trade; Wi-Fi and Bluetooth&reg; connectivity devices, XMC&trade; Industrial MCUs, and EZ-USB&trade;/EZ-PD&trade; wired connectivity controllers. ModusToolbox&trade; incorporates a comprehensive set of BSPs, HAL, libraries, configuration tools, and provides support for industry-standard IDEs to fast-track your embedded application development

<br>


## Other resources

Infineon provides a wealth of data at [www.infineon.com](https://www.infineon.com) to help you select the right device, and quickly and effectively integrate it into your design.


## Document history

Document title: *CE241521* – *PSOC&trade; Edge MCU:  Encrypted boot*

 Version | Description of change
 ------- | ---------------------
 1.x.0   | New code example <br> Early access release
 2.0.0   | GitHub release
 2.0.1   | Updated README instructions
<br>


All referenced product or service names and trademarks are the property of their respective owners.

The Bluetooth&reg; word mark and logos are registered trademarks owned by Bluetooth SIG, Inc., and any use of such marks by Infineon is under license.

PSOC&trade;, formerly known as PSoC&trade;, is a trademark of Infineon Technologies. Any references to PSoC&trade; in this document or others shall be deemed to refer to PSOC&trade;.

---------------------------------------------------------

© Cypress Semiconductor Corporation, 2025. This document is the property of Cypress Semiconductor Corporation, an Infineon Technologies company, and its affiliates ("Cypress").  This document, including any software or firmware included or referenced in this document ("Software"), is owned by Cypress under the intellectual property laws and treaties of the United States and other countries worldwide.  Cypress reserves all rights under such laws and treaties and does not, except as specifically stated in this paragraph, grant any license under its patents, copyrights, trademarks, or other intellectual property rights.  If the Software is not accompanied by a license agreement and you do not otherwise have a written agreement with Cypress governing the use of the Software, then Cypress hereby grants you a personal, non-exclusive, nontransferable license (without the right to sublicense) (1) under its copyright rights in the Software (a) for Software provided in source code form, to modify and reproduce the Software solely for use with Cypress hardware products, only internally within your organization, and (b) to distribute the Software in binary code form externally to end users (either directly or indirectly through resellers and distributors), solely for use on Cypress hardware product units, and (2) under those claims of Cypress's patents that are infringed by the Software (as provided by Cypress, unmodified) to make, use, distribute, and import the Software solely for use with Cypress hardware products.  Any other use, reproduction, modification, translation, or compilation of the Software is prohibited.
<br>
TO THE EXTENT PERMITTED BY APPLICABLE LAW, CYPRESS MAKES NO WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, WITH REGARD TO THIS DOCUMENT OR ANY SOFTWARE OR ACCOMPANYING HARDWARE, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE.  No computing device can be absolutely secure.  Therefore, despite security measures implemented in Cypress hardware or software products, Cypress shall have no liability arising out of any security breach, such as unauthorized access to or use of a Cypress product. CYPRESS DOES NOT REPRESENT, WARRANT, OR GUARANTEE THAT CYPRESS PRODUCTS, OR SYSTEMS CREATED USING CYPRESS PRODUCTS, WILL BE FREE FROM CORRUPTION, ATTACK, VIRUSES, INTERFERENCE, HACKING, DATA LOSS OR THEFT, OR OTHER SECURITY INTRUSION (collectively, "Security Breach").  Cypress disclaims any liability relating to any Security Breach, and you shall and hereby do release Cypress from any claim, damage, or other liability arising from any Security Breach.  In addition, the products described in these materials may contain design defects or errors known as errata which may cause the product to deviate from published specifications. To the extent permitted by applicable law, Cypress reserves the right to make changes to this document without further notice. Cypress does not assume any liability arising out of the application or use of any product or circuit described in this document. Any information provided in this document, including any sample design information or programming code, is provided only for reference purposes.  It is the responsibility of the user of this document to properly design, program, and test the functionality and safety of any application made of this information and any resulting product.  "High-Risk Device" means any device or system whose failure could cause personal injury, death, or property damage.  Examples of High-Risk Devices are weapons, nuclear installations, surgical implants, and other medical devices.  "Critical Component" means any component of a High-Risk Device whose failure to perform can be reasonably expected to cause, directly or indirectly, the failure of the High-Risk Device, or to affect its safety or effectiveness.  Cypress is not liable, in whole or in part, and you shall and hereby do release Cypress from any claim, damage, or other liability arising from any use of a Cypress product as a Critical Component in a High-Risk Device. You shall indemnify and hold Cypress, including its affiliates, and its directors, officers, employees, agents, distributors, and assigns harmless from and against all claims, costs, damages, and expenses, arising out of any claim, including claims for product liability, personal injury or death, or property damage arising from any use of a Cypress product as a Critical Component in a High-Risk Device. Cypress products are not intended or authorized for use as a Critical Component in any High-Risk Device except to the limited extent that (i) Cypress's published data sheet for the product explicitly states Cypress has qualified the product for use in a specific High-Risk Device, or (ii) Cypress has given you advance written authorization to use the product as a Critical Component in the specific High-Risk Device and you have signed a separate indemnification agreement.
<br>
Cypress, the Cypress logo, and combinations thereof, ModusToolbox, PSoC, CAPSENSE, EZ-USB, F-RAM, and TRAVEO are trademarks or registered trademarks of Cypress or a subsidiary of Cypress in the United States or in other countries. For a more complete list of Cypress trademarks, visit www.infineon.com. Other names and brands may be claimed as property of their respective owners.