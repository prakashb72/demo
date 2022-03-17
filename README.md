# XmlCli Reference Script

XmlCli Reference scripts which are based on XML Information & BIOS CLI interface protocol.

These reference scripts provides several capabilities including but not limited to:
>- Programming/Reading BIOS knobs
>- Fetching Platform XML from target
>- Flashing BIOS
>- Fetching BIOS
>- Ucode read
>- Save & update
>- System information

These scripts are generic and platform/program independent (as long as BIOS on SUT BIOS supports XML CLI interface).

Please let us ([xmlcli](mailto:xmlcli@intel.com)) know for any queries or feedback related to these scripts/interface.
For more questions/clarification visit this link [questions at stackoverflow.intel.com](https://stackoverflow.intel.com/questions/tagged/5583)
For any general help or query related to `xmlcli`, you can also post or look at community at: [QA](https://stackoverflow.intel.com/) (make sure to add tag: `xmlcli` while posting question)

---

## Supported Interface Types

- Legacy ITP
- ITP II
- Lauterbac
- DCI
- Simics
- SVOS
- Windows
- LINUX
- EFI Shell
- SV HIF card
- TSSA
- Offline mode (or stub mode, to enable BIOS/IFWI Editing)

## Prerequisites

- [Python](https://www.python.org/downloads/) software
- `CCB SDK` (for HWAPI interface) for running it on `Windows SUT`

## Common importing steps

Copy the Reference scripts on any folder on the system

open Python Prompt and run following commands

```python
import sys
sys.path.append(r"<Path_To_XmlCliRefScripts>")  # append path to reference scripts
from src import XmlCli as cli
```

## Setting Interface Type

Need to select the interface to indicate the scripts on which access method to use (depending on which environment we expect the script to operate in).

```python
from src import XmlCli as cli
cli.clb._setCliAccess("<access-method>")
```

Below are listed valid `<access-method>`:

| Access Method | Remarks |
| --- | --- |
| `itpii` | For using ITPII as interface, need to have ITP S/W & H/W installed on the system
| `svlegitp` | For using Legacy ITP as interface, need to have Legacy ITP S/W & H/W installed on the system
| `ltb` | For using Lauterbac as interface, need to have LTB S/W & H/W installed on the system
| `dci` | For using dci as interface, need to have ITP & DCI S/W & H/W installed on the system
| `itpsimics` | For using simics as interface (Pre-Silicon), need to have simics S/W installed on the system. refer [BKM for enabling](https://stackoverflow.intel.com/articles/11518)
| `svos` | For using svos as interface, need to have svos disk installed on the system
| `winssa` | For using ssa as primary interface on Windows (uses HWAPI DLL), need to have SSA folder copied under `C:\Python27\Lib\site-packages\` SSA folder can be unzipped from `\\bascrd101\svshare\Amol\XmlCliRefScripts\ssa.zip`
| `winhwa` | For using standalone hwapi (uses HWAPI DLL), does not need any special installation
| `winsdk` | (recommended) For using SDK as Windows interface (uses HWAPI DLL), need to have "Intel® CCG Platform Tools SDK" installed on the system. Not using Latest SDK may result in BSOD, so we recommend to get it from [Advanced System Tools sharepoint](https://intel.sharepoint.com/sites/AdvancedSystemTools/Shared%20Documents/Forms/AllItems.aspx?viewid=13e7e46a-5974-4b3f-9c55-e0960107e28f), For more details refer “Advanced System Tools” : [Wiki](https://wiki.ith.intel.com/display/adotools/Advanced+System+Tools+and+Analytics)
| `winrwe` | For using `RW.exe` as Windows interface (**slow**, least recommended)
| `linux` | For using linux as interface, need to open Python Prompt in root permissions.
| `esxi` | Port for the VMware hypervisor, formally name `ESXi`, need to open Python prompt with root permission, (driver is auto-enabled when interface initialized)
| `uefi` | For using EFI SHELL as interface, need to have EFI SHELL's Python Interpreter on the system
| `svhif` | For using SV Host Interface Card (Dishon, Kfir) as interface, need to have Sv HIF card plugged on the system and Sv Mode initialized.
| `tssa` | For using Windows, Mac & Linux as Interface, need to have SSA pkg installed that support baseaccess python wrapper.

## Running popular commands

After initializing the desired interface, the use may run following commands.

If the commands `return 0`, it means the operation was `successful`, else there was an error.

### Standard import steps

```python
import sys
sys.path.append(r"<Path_To_XmlCliRefScripts>")
from src import XmlCli as cli
cli.clb._setCliAccess("itpii") # Set desired Interface (for example itpii)
```

### Step to check XmlCli capability on current System

```python
cli.clb.ConfXmlCli()  # Check if XmlCli is supported &/ Enabled on the current system.
```

| Return value of `cli.clb.ConfXmlCli` | Meaning |
| -- | -- |
| 0 | XmlCli is already **supported & enabled**. |
| 1 | XmlCli is **not supported** on the current BIOS or the System BIOS has not completed Boot. |
| 2 | XmlCli is **supported** but was **not enabled**, the script has now enabled it and **SUT needs reboot** to make use of XmlCli. |

### To Save Target XML file

```python
from src import XmlCli as cli
cli.savexml() # Save Target XML as <Path_To_XmlCliRefScripts>\out\PlatformConfig.xml File.
cli.savexml(r"<MyFilePath>")  # Save Target XML as <MyFilePath> File.
cli.savexml(0, r"<MyBIOS_or_IFWI_FilePath>")  # Extract the XML data from desired BIOS or IFWI binary. Will Save Target XML in <Path_To_XmlCliRefScripts>\out folder.
cli.savexml(r"<MyFilePath>", r"<MyBIOS_or_IFWI_FilePath>")  # Extract the XML data from desired BIOS or IFWI binary. Will Save Target XML as <MyFilePath>.
```

### Usage instruction for IPClean (External) BIOS/IFWI
XmlCli now supported in IPClean BIOS image with below capabilities:
1. API related to NVRAM Variables such as: `CvProgKnobs`, `CvReadKnobs`, `CvLoadDefaults`, `CvRestoreModify`, `savexml`
2. In order to execute these APIs it requires to set `cli.clb.AuthenticateXmlCliApis = True` for the session.
3. These Python Reference Scripts for SDL approval is work in progress, so as such **it must be used for Intel Internal only**.

### To Read BIOS settings

> For **Online** command to run successfully, the target must complete BIOS boot.
> For **Offline** mode, you need to pass the link to BIOS or IFWI binary.

- `Knob_A` & `Knob_B` in the below examples are the knob names taken from the `name` attribute from the `<biosknobs>` section in the XML, it is **case sensitive**.

  ```python
  from src import XmlCli as cli
  cli.CvReadKnobs("Knob_A=Val_1, Knobs_B=Val_2") # Reads the desired Knob_A & Knob_B settings from the SUT and verifies them against Val_1 & Val_2 respectively.
  cli.CvReadKnobs()  # same as above, just that the Knob entries will be read from the default cfg file (<Path_To_XmlCliRefScripts>\cfg\BiosKnobs.ini).
  # the default cfg file pointer can be programed to desired cfg file via following command.
  cli.clb.KnobsIniFile = r"<MyCfgFilePath>"
  cli.CvReadKnobs("Knob_A=Val_1, Knobs_B=Val_2", r"<MyBIOS_or_IFWI_FilePath>")  # Reads & verifies the desired knob settings from the given BIOS or IFWI binary.
  cli.CvReadKnobs(0, r"<MyBIOS_or_IFWI_FilePath>") # same as above, just that the Knob entries will be read from the cli.clb.KnobsIniFile cfg file instead.
  ```

### To Program BIOS settings

> For **Online** command to run successfully, the target must complete BIOS boot.
> For **Offline** mode, you need to pass the link to BIOS or IFWI binary.

- `Knob_A` & `Knob_B` in the below examples are the knob names taken from the `name` attribute from the `<biosknobs>` section in the XML, it is **case sensitive**.

```python
from src import XmlCli as cli
cli.CvProgKnobs("Knob_A=Val_1, Knobs_B=Val_2")  # Programs the desired Knob_A & Knob_B settings on the SUT and verifies them against Val_1 & Val_2 respectively.
cli.CvProgKnobs()  # same as above, just that the Knob entries will be Programed from the default cfg file (<Path_To_XmlCliRefScripts>\cfg\BiosKnobs.ini).
# the default cfg file pointer can be programed to desired cfg file via following command.
cli.clb.KnobsIniFile = r"<MyCfgFilePath>"
cli.CvProgKnobs("Knob_A=Val_1, Knobs_B=Val_2", r"<MyBIOS_or_IFWI_FilePath>")  # Program the desired knob settings as new default value, operates on BIOS or IFWI binary, new BIOS or IFWI binary will be generated with desired settings.
cli.CvProgKnobs(0, r"<MyBIOS_or_IFWI_FilePath>")  # same as above, just that the Knob entries will be Programed from the cli.clb.KnobsIniFile cfg file instead.

# To Load Default BIOS settings on the SUT. Offline mode not supported or not Applicable.
cli.CvLoadDefaults()  # Loads/Restores the default value back on the system, also shows which values were restored back to its default Value.
```

### To Program only desired BIOS settings and reverting rest all settings back to its default value

> **Offline** mode not supported or not Applicable.

- `Knob_A` & `Knob_B` in the below examples are the knob names taken from the `name` attribute from the `<biosknobs>` section in the XML, it is **case sensitive**.

```python
from src import XmlCli as cli
cli.CvRestoreModifyKnobs("Knob_A=Val_1, Knobs_B=Val_2")  # Programs the desired Knob_A & Knob_B settings and restores everything else back to its default value.
cli.CvRestoreModifyKnobs()  # same as above, just that the Knob entries will be Programed from the cli.clb.KnobsIniFile cfg file instead.
```

> Offline editing of BIOS will update FV_BB section of BThis could be an issue with Secure Boot profiles (Profile 5 IFWI images)

To make sure offline Edited BIOSes for Knob changes boot fine with Secure Profile IFWI's you need to supply the re-signing script/pkg
See below for one such example and steps to be used for TGL BIOSes.

TGL Re-signing Pkg (Maintained by dakota.chiang@intel.com) can be found under `\\bascrd101.gar.corp.intel.com\SVShare\amol\XmlCliRefScripts\Profile_5\ResignIbbForBtG_TGL.zip`

```python
from src import XmlCli as cli
cli.fwp.SecureProfileEditing = True   # if not set to True, Re-Signing Process will be skipped
cli.fwp.ReSigningFile = r'C:\Users\ashinde\Desktop\MyFolder\Profile5\ResignIbbForBtG_TGL\ResignIbbForBtG.bat'     # by default this variable is empty, please populate this variable with Re-Signing Script File Ptr
cli.CvProgKnobs("BootFirstToShell=1, EfiNetworkSupport=3", r'<BIOS_OR_IFWI_File_PTR>')
```

## SPI Operations

### Process Ucode operation
> For **Offline** mode, you need to pass the link to BIOS or IFWI binary

```python
from src import XmlCli as cli
cli.ProcessUcode() # Online mode, Operates on SUT, by default reads the current Ucode entries.
cli.ProcessUcode("read", r"<MyBIOS_or_IFWI_FilePath>")  # Offline mode, Operates on given BIOS or IFWI binary file, read the current Ucode entries.

cli.ProcessUcode("update", 0, r"<Desired_Ucode_File>")  # Online mode, Operates on SUT, Updates the desired Patch file (.inc or .pdb format) on the target system.
cli.ProcessUcode("update", r"<MyBIOS_or_IFWI_FilePath>", r"<Desired_Ucode_File>")  # Offline mode, same as above except that it Operates on given BIOS or IFWI binary file, new BIOS/IFWI binary will be generated. The desired Ucode file argument can also be list of Ucode files for bulk "update" operation, see below for examples.
cli.ProcessUcode("update", 0, [r"<Desired_Ucode_File1>", r"<Desired_Ucode_File2>"])
cli.ProcessUcode("update", r"<MyBIOS_or_IFWI_FilePath>", [r"<Desired_Ucode_File1>", r"<Desired_Ucode_File2>"])

cli.ProcessUcode("save", ReqCpuId=CpuIdVal)  # CpuIdVal as integer; Online mode, Saves the Ucode data as a file for given CpuId entry under <Path_To_XmlCliRefScripts>\out folder.
cli.ProcessUcode("save", r"<MyBIOS_or_IFWI_FilePath>, ReqCpuId=0x<CpuIdVal>")  # Offline mode, same as above except that it Operates on given BIOS or IFWI binary file, new BIOS/IFWI binary will be generated.

cli.ProcessUcode("saveall")  # Online mode, Saves the Ucode data as a file for all entries under <Path_To_XmlCliRefScripts>\out folder.
cli.ProcessUcode("saveall", r"<MyBIOS_or_IFWI_FilePath>")  # Offline mode, same as above except that it Operates on given BIOS or IFWI binary file, new BIOS/IFWI binary will be generated.

cli.ProcessUcode("delete", ReqCpuId=0x<CpuIdVal>)  # Online mode, Deletes the Ucode entry for given CpuId entry.
cli.ProcessUcode("delete", r"<MyBIOS_or_IFWI_FilePath>, ReqCpuId=0x<CpuIdVal>")  # Offline mode, same as above except that it Operates on given BIOS or IFWI binary file, new BIOS/IFWI binary will be generated.

cli.ProcessUcode("deleteall")  # Online mode, Deletes all the Ucode entries.
cli.ProcessUcode("deleteall", r"<MyBIOS_or_IFWI_FilePath>, ReqCpuId=0x<CpuIdVal>")  # Offline mode, same as above except that it Operates on given BIOS or IFWI binary file, new BIOS/IFWI binary will be generated.

cli.ProcessUcode("fixfit")  # Online mode, Operates on SUT, Verifies FIT, and if FIT table was wrong, fixes it.
cli.ProcessUcode("fixfit", r"<MyBIOS_or_IFWI_FilePath>")  # Offline mode, same as above except that it Operates on given BIOS or IFWI binary file, new BIOS/IFWI binary will be generated.
```

#### For resiliency bios or the use case wherein slot size of ucode (microcode) needs not to resized

If user wish to provide new ucode with size less then the existing size of ucode, and wishes to keep slot size intact to not realign offset additional flag `Resiliency` would help to achieve the goal.
If parameter is set to value `True` scripts will not perform implicit FixFit operation and also add padding to maintain slot size.

Example:
```python
from src import XmlCli as cli
cli.ProcessUcode("update", r"<MyBIOS_or_IFWI_FilePath>", [r"<Desired_Ucode_File1>", r"<Desired_Ucode_File2>"], Resiliency=True)
```

### SPI Flash Read/Write

Run following Commands to enable Flashing support.

> Please note BIOS needs to support Flash update XmlCli API's for this to work.

1. Enable SPI Flashing support on CCG BIOS

  ```python
  cli.CvProgKnobs("PchRtcLock=0,FlashWearOutProtection=0,BiosGuard=0,FprrEnable=0,PchBiosLock=0")
  ```
  or
  ```python
  cli.CvProgKnobs("PchRtcMemoryLock=0,BiosGuard=0,FprrEnable=0,PchBiosLock=0")
  ```

2. Reboot the system.

3. After running this confirm that the settings are set correctly, by running following command

```python
cli.CvReadKnobs("PchRtcLock=0,FlashWearOutProtection=0,BiosGuard=0,FprrEnable=0,PchBiosLock=0")
```
or
```python
cli.CvReadKnobs("PchRtcMemoryLock=0,BiosGuard=0,FprrEnable=0,PchBiosLock=0")
# The above command should return 0 (0 = Success).
```

#### For flashing the SPI run one of following command

```python
from src import XmlCli as cli
cli.SpiFlash('write', r"<file>", RegionOffset=Offset)  # to Flash SPI Region from Offset to Offset+Sizeof(file)
```

#### For Reading the SPI flash run one of following commands

```python
from src import XmlCli as cli
cli.SpiFlash('read', r"<file_To_Save>", RegionOffset=Offset, RegionSize=Size)  # to fetch SPI Region from Offset to Offset+Size and save it to the given file
```

> Note: SPI `read` operation doesn't need a reboot of the system, but if you are using `write` operation, its desired to reboot you system after its done. Please verify that your new IFWI or BIOS ROM is flashed by going into the setup page and reading the BIOS version.

### Add MSR & IO Read/Write CLI functions (`Only for DCG`)

#### Usage

```python
from src import XmlCli as cli
cli.IoAccess(operation, IoPort, Size, IoValue)
cli.MsrAccess(operation, MsrNumber, ApicId, MsrValue)
```

#### Example

```python
from src import XmlCli as cli
cli.IoAccess(cli.clb.IO_WRITE_OPCODE, 0x84, 1, 0xFA)
cli.IoAccess(cli.clb.IO_READ_OPCODE, 0x84, 1)
cli.MsrAccess(cli.clb.READ_MSR_OPCODE, 0x53, 0)
cli.MsrAccess(cli.clb.WRITE_MSR_OPCODE, 0x1A0, 0, 0x1)
```

## Execution under EFI Shell

### Usage Instructions to run `Python.efi` from EFI shell

- Download and unzip the [Python EFI Package](https://wiki.ith.intel.com/pages/viewpage.action?pageId=1296008120) (Developed and Supported by CCG WSS ADO Advanced System Tools)
- Copy the whole `EFI` folder to `USB Stick` [**MAKE SURE: you do copy the whole EFI folder at the root of the file system**]
- Insert USB Stick to the DUT (Device Under Test)
- Boot to EFI shell
- If USB Stick detected as `FS0:` file system, Type `FS0:` on the shell prompt to get `FS0:>` prompt string on the shell
- From the shell prompt issue the following command `Python.efi -$` to get into interactive mode of interpreter (Note: if interpreter does not launch then try to run it from EFI\Tools directory where `Python.efi` located under the copied EFI folder)
- If you want to run any script from file say `MyScript.py`, Copy the script to USB Stick run the following command `Python.efi -$ <pathToScriptDirectoryonUSBStick>\MyScript.py`

### To enable XmlCli Python reference Scripts support on EFI Shell

In the above steps please following as an additional step:

- Copy XmlCli reference Scripts folder under EFI folder of the USB Stick
- You can now import the XmlCli scripts and make use of it as usual, please note the `from src import XmlCli as cli` will automatically detect the UEFI as interface Type.

  ```python
  import sys
  sys.path.append(r"<FS0:\EFI\XmlCli>")
  from src import XmlCli as cli
  cli.savexml()
  cli.CvReadKnobs("Knob_A=Val_1, Knob_B=Val_2")
  cli.CvProgKnobs("Knob_A=Val_1, Knob_B=Val_2")
  ```

## Usage for `eBiosSetupPage.py` - XmlCli's Virtual BIOS Setup Page Gui

> Prerequisite for online mode - BIOS Knobs `XmlCliSupport` and `PublishSetupPgPtr` needs to be set to enable

- open python prompt and run gui

  ```python
  from modules.eBiosSetupPage import gen_gui  # import method which generates gui
  gen_gui()  # generates GUI to launch the virtual setup page
  ```

- Select valid options in GUI prompt

  - working mode

    - `online`
      - for host system
      - select valid access method for online mode from GUI dropdown menu

    - `offline`
      - for xml/bin/rom file

  - Publish all knobs
    - user can select whether to publish all knobs or not (applicable for **online** mode, and for **bin/rom** file for offline mode)

> Note: For XML or BIOS binary File as input All knobs will be published

#### Prefix of dropdown Knob options selection values in GUI are as below

| Prefix | Interpretation |
| --- | --- |
| ◇ | Default Value of knob |
| ◈ | Current value of knob |
| ◆ | current and default value of knob |

#### Interpretation of buttons on Virtual Setup Page GUI

| Button | Interpretation |
| --- | --- |
| Push Changes | Apply changes to system if online mode else to `bin/rom` file, (N/A for offline xml) |
| View Changes | View saved changes in new window |
| Exit | Exit the GUI |
| Reload | Reload the GUI |
| Discard Changes | Discard any change made, any value if modified are restored to current value (`CurrentVal`) of knob xml file |
| Load Defaults | Restore to default values and revert any changes made |

#### Status of buttons and action on Virtual Setup Page GUI on various modes/scenarios

| Button          | Online | Offline `.xml` | Online `.xml` | `.bin` or `.rom` file |
| --------------- | ------ | --- | --- | --- |
| Push Changes    | ✔  changes directly written to the SUT | :x: | ❌ (if _Publish all_ selected in previous option) | :heavy_check_mark: changes written to new bin file |
| View Changes    | ✔     | ✔    | ✔     | ✔    |
| Exit            | ✔     | ✔    | ✔     | ✔    |
| Reload          | ✔     | ✔    | ✔     | ✔    |
| Discard Changes | ✔     | ✔    | ✔     | ✔    |
| Load Defaults   | ✔     | ❌    | ❌    | ❌    |


## EFI shell Command sequence to Modify PTT Present/TPM Device via XmlCli

Aim: To modify PTT Present to set to desire value - `<value>` which could be `0` - disable or `1` - enable

> Applicable only if you are using Client BIOS platform AlderLake or beyond,
> Copy `XmlCliKnobs.efi` to pendrive (or any file system detectable in UEFI Shell) from latest XmlCli reference scripts.

| Command | Description |
| ------- | ----------- |
| `XmlCliKnobs.efi CX` | Ensure XmlCli is enabled, if it's not enable, this command helps to enable XmlCli, Reboot SUT if XmlCli was not enabled. |
| `XmlCliKnobs.efi APL -s "EndOfPostMessage=0"` | End of Post must be disabled for the APP to be able to send HECI messages to toggle PTT setting. this step programs the `EndOfPostMessage` knob to `0` (disable) |
| `XmlCliKnobs.efi ROL -s "EndOfPostMessage=0"` | Ensure the BIOS knob `EndOfPostMessage` value still disabled (`0`) by reading it back |
| `reset -c` | Perform a cold reset via EFI shell |
| `XmlCliKnobs.efi TPM <value>` | Send heci Message to modify PttPresent |
| `XmlCliKnobs.efi APL -s "PttPresent=<value>, EndOfPostMessage=2"` | When SUT boots back to shell after the reset, HECI messaging will now be allowed, run these SHELL command to Enable PTTPresent setting, also ensure we revert the EndOfPostMessage knob to its default value (our intent on toggling `PTTPresent` is now met). |
| `XmlCliKnobs.efi ROL -s "PttPresent=<value>, EndOfPostMessage=2"` | Read the bios knobs settings to ensure the knob values were updated as per above step. |
| `reset -c` | Perform a cold reset via EFI shell, cold reset is must for the changes to take effect |


## Usage instruction for Uefi Parser

The parser is able to generate the json representation from BIOS or IFWI image.

Key features:
-   JSON representation, lightweight database with keys and values with ease of readability
-   Works with both SUT and offline image

Working with SUT:

Below command to be executed only after enabling applicable access method:

```python
from src import XmlCli as cli
max_bios_size = 12 * (1024**2)  # 12 MB - configure based on the platform used
# If max_bios_size argument is not specified then by default it uses 32 MB dump to lookup for BIOS image
bios_image = cli.clb.get_bin_file("winhwa", max_bios_size=max_bios_size)  # variable will have location of bios image dump stored from memory
```

Initiate the Parsing with below commands:

```python
from src.common import bios_fw_parser

bios_image = "absolute-path/to/bios-image.rom"

uefi_parser = bios_fw_parser.UefiParser(bin_file=bios_image,  # binary file to parse
                                parsing_level=0,  # parsing level to manage number of parsing features
                                base_address=0,  # (optional) provide base address of bios FV region to start the parsing (default 0x0)
                                guid_to_store=[]  # if provided the guid for parsing then parser will look for every GUID in the bios image
                                )
# parse binary
output_dict = uefi_parser.parse_binary()
output_dict = uefi_parser.sort_output_fv(output_dict)  # (optional) only to sort output by FV address
# write content to json file
output_file = "absolute-path/to/output.json"
uefi_parser.write_result_to_file(output_file, output_dict=output_dict)
# Below code block is only to store map result to json for FV region(s) extracted by guid lookup
if uefi_parser.guid_to_store:
    # additional test for GUIDs to store
    result = uefi_parser.guid_store_dir  # result of passed guid
    user_guid_out_file = "absolute-path/to/guid-stored/output.json"
    # Store guid stored result to json file
    uefi_parser.write_result_to_file(user_guid_out_file, output_dict=uefi_parser.stored_guids)
```

### MEBx Validation Protocol Usage

These APIs are available to Intel Internal only projects and can only be accessed through EFI APP released with these
scripts.

> Before running any command API you need to ensure value of `EndOfPostMessage` to `0` and reset platform.
>
> 1. Ensuring `EndOfPostMessage` value to disable state `XmlCliKnobs.efi AP -s "EndOfPostMessage=0"`
>   - if you perform this command then, after the usage completion it also needs to be reverted to it's original state value `2`
>     by executing `XmlCliKnobs.efi AP -s "EndOfPostMessage=2"`
> 2. Reset Platform

| API Name | Syntax |
| -------- | ------ |
| Enable AMT | `XmlCliKnobs.efi MEBX AP EnableAMT` |
| Disable AMT | `XmlCliKnobs.efi MEBX AP DisableAMT` |
| Update Host domain name | `XmlCliKnobs.efi MEBX AP SetHostDomainName <required-host-domain-name>`|
| UnProvisioning AMT State | `XmlCliKnobs.efi MEBX AP UnprovisioingAMTState`|
| Configuring PKI DNS suffix state | `XmlCliKnobs.efi MEBX AP PKISuffixUpdate <domain.name>` |


### Using Logging in XmlCli for debugging

Importing module:

```python
from src.common import logger

settings = logger.Setup(  # All setup arguments are optional/ if not specified default value will be overridden
        log_level="DEBUG",  # default level would be utils.LOG_LEVEL
        log_title="XmlCli",  # default to utils.LOGGER_TITLE (Avoid overriding this!!!)
        sub_module="XmlCliLib",  # To allow printing submodule name to logging, for better lookup to filter
        log_dir="absolute/path/to/log/directory/",  # defaults to utils.LOG_DIR
        log_file_name="name_of_log_file.log",  # defaults to title and timestamp
        write_in_file=True,  # Specifies whether to write the log in file or not defaults to utils.FILE_LOG
        print_on_console=True,  # Specifies whether to write log on console or not defaults to utils.CONSOLE_STREAM_LOG
    )

# To display config of system run below function:
settings.display_system_configs()

# Detailed logging for function entry point:
settings.display_system_configs()

# To log whether your script enters and exit to function along with the arguments feed to the function, add below line overridden any function/method:
@settings.log_function_entry_and_exit
def my_function(param1, param2):
    pass
```

This will log the details on when your script enters to function: `my_function`, lists the parameters and also logs when the script exits from the function scope.

Example:
```python
@settings.log_function_entry_and_exit
def ProcessUcode(Operation='READ', BiosBinaryFile=0, UcodeFile=0, ReqCpuId=0, outPath='', BiosBinListBuff=0, ReqCpuFlags=0xFFFF, PrintEn=True):
    # rest of the function definition..
    # calculation, etc.
    pass
```

logging individually as:

```python
log = settings.logger
log.debug("log message")
log.info("log message")
log.error("error message")
```
## **Perform Startup ACM Patching**

This feature allows user to patch startup ACM binary (Type 2) on input IFWI/BIOS binary
  - Run following commands to perform the startup ACM patching

### **To Read the information about the Startup ACM binary inside the IFWI/BIOS**
Syntax:
```python
# Performs Read operation and displays ACM related information. Extracted ACM binary will be saved at <absolute/path/to/output.bin>
cli.ProcessAcm(operation="read",binary_file=r"<absolute/path/to/ifwi-or-bios.bin>", startup_acm_file=0, output_path=r"<absolute/path/to/acm.bin>")

cli.ProcessAcm("read",r"<absolute/path/to/ifwi-or-bios.bin>", 0,r"<absolute/path/to/acm.bin>")
```
```python
# Performs Read operation and displays ACM related information. Will save the Extracted ACM binary at default location: <xmlcli-out-directory>\ACM_binary.bin
cli.ProcessAcm(operation="Read", binary_file=r"<absolute/path/to/ifwi-or-bios.bin>")

cli.ProcessAcm("Read",r"<absolute/path/to/ifwi-or-bios.bin>")
```


### **To Perform the Startup ACM patching on IFWI/BIOS** <br>
Syntax:
```python
# Performs ACM patching. Will save the Patched IFWI/BIOS at default location: <xmlcli-out-directory>\ACM_PatchedBios.rom


cli.ProcessAcm(operation="Update", binary_file=r"<absolute/path/to/ifwi-or-bios.bin>", startup_acm_file=r"<absolute/path/to/acm.bin>")

or

cli.ProcessAcm("Update",r"<absolute/path/to/ifwi-or-bios.bin>",r"<absolute/path/to/acm.bin>")
```
```python
# Performs ACM patching. Will save the patched IFWI/BIOS as <absolute/path/to/output.bin>

cli.ProcessAcm(operation="Update", binary_file=r"<absolute/path/to/ifwi-or-bios.bin>", startup_acm_file=r"<absolute/path/to/acm.bin>", output_path=r"<absolute/path/to/output.bin>")

or

cli.ProcessAcm("Update", r"<absolute/path/to/ifwi-or-bios.bin>", r"<absolute/path/to/acm.bin>", r"<absolute/path/to/output.bin>")

```
> For now `cli.ProcessAcm()` API works only for the BIOS/IFWI binary having only one Startup ACM Entry (Type 2 Entry)
![image](https://github.com/prakashb72/applications.validation.platform-automation.xmlcli.xmlcli/assets/89830973/96b68269-e238-4467-8ee3-525c4cec6328)

### **Perform Startup ACM Patching**

This feature allows user to read or update startup ACM binary (Type 2) on given input IFWI/BIOS binary
  - Run following commands to perform the startup ACM patching

#### **To Read the information about the Startup ACM binary inside the IFWI/BIOS**

```python
from src import XmlCli as cli
# Performs Read operation and displays ACM related information.
# Extracted ACM binary will be saved at <absolute/path/to/output.bin>
cli.ProcessAcm("read", r"<absolute/path/to/ifwi-or-bios.bin>", 0, r"<absolute/path/to/acm.bin>")
cli.ProcessAcm(operation="read", binary_file=r"<absolute/path/to/ifwi-or-bios.bin>", startup_acm_file=0, output_path=r"<absolute/path/to/acm.bin>")

# Performs Read operation and displays ACM related information.
# Will save the Extracted ACM binary at default location: <xmlcli-out-directory>\ACM_binary.bin
cli.ProcessAcm("Read", r"<absolute/path/to/ifwi-or-bios.bin>")
cli.ProcessAcm(operation="Read", binary_file=r"<absolute/path/to/ifwi-or-bios.bin>")
```

#### **To Perform the Startup ACM patching on IFWI/BIOS** <br>

```python
from src import XmlCli as cli
# Performs ACM patching.
# Will save the Patched IFWI/BIOS at default location: <xmlcli-out-directory>\ACM_PatchedBios.rom
cli.ProcessAcm("Update", r"<absolute/path/to/ifwi-or-bios.bin>", r"<absolute/path/to/acm.bin>")
cli.ProcessAcm(operation="Update", binary_file=r"<absolute/path/to/ifwi-or-bios.bin>", startup_acm_file=r"<absolute/path/to/acm.bin>")

# Performs ACM patching.
# Will save the patched IFWI/BIOS as <absolute/path/to/output.bin>
cli.ProcessAcm("Update", r"<absolute/path/to/ifwi-or-bios.bin>", r"<absolute/path/to/acm.bin>", r"<absolute/path/to/output.bin>")
cli.ProcessAcm(operation="Update", binary_file=r"<absolute/path/to/ifwi-or-bios.bin>", startup_acm_file=r"<absolute/path/to/acm.bin>", output_path=r"<absolute/path/to/output.bin>")
```

> For now `cli.ProcessAcm()` API works only for the BIOS/IFWI binary having only one Startup ACM Entry (Type 2 Entry)
![image](https://github.com/prakashb72/applications.validation.platform-automation.xmlcli.xmlcli/assets/89830973/96b68269-e238-4467-8ee3-525c4cec6328)

------------------------------------------------------------------------------------------------------------------------

## Please refer below table for Respective document and purpose

| Wiki/Documentation | Applicable on IPClean BIOS? |
| ------------------ | --------------------------- |
| [PreRequisites](https://stackoverflow.intel.com/articles/8947) | Yes |
| [CheatSheet](https://stackoverflow.intel.com/articles/7938) | Yes |
| [Collecting setup knob difference of 2 BIOS/IFWI](https://stackoverflow.intel.com/articles/12791) | Yes |
| [Access Interfaces](https://stackoverflow.intel.com/articles/17435) | Yes |
| [Commands for using EFI APP](https://stackoverflow.intel.com/articles/9013) | Yes |
| [Using XmlCli in IPCLEAN BIOS](https://stackoverflow.intel.com/articles/24233) | Yes |
| [SPI Flashing](https://stackoverflow.intel.com/articles/8945) | **No** |
| [Using XmlCli on EFI Python environment](https://stackoverflow.intel.com/articles/17051) | **No** |
| [Consuming XmlCli for Configuring TXT settings](https://stackoverflow.intel.com/articles/26869) | **No** |
| [Modifying TPM Values](https://stackoverflow.intel.com/articles/19949) | **No** |
| [Enable/Disable SecureBoot from EFI Shell using Python EFI](https://stackoverflow.intel.com/articles/17245) | **No** |
| [Configuring MEBx AMT Settings](https://stackoverflow.intel.com/articles/24154) | **No** |

------------------------------------------------------------------------------------------------------------------------

**File a Feature Request at: <http://goto/xmlcli-feature-request>**
**File a bug at: <http://goto/xmlcli-bug>**
For getting ***[release notification](https://eclists.intel.com/sympa/subscribe/xmlcli)***, subscribe at: <https://eclists.intel.com/sympa/subscribe/xmlcli> (use intel email id only, other emails will be discarded automatically and won’t be added to subscription)
**Other Helpful Links Related to XmlCli:**
- [xmlcli-resources](https://goto.intel.com/xmlcli-resources)
- [xmlcli-releases](https://goto.intel.com/xmlcli-releases)
- [xmlcli-faq](https://goto.intel.com/xmlcli-faq)

