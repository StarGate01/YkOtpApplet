# VivoKey fork of YkOtpApplet

Parent repository: https://github.com/arekinath/YkOtpApplet by Alex Wilson
## About

This is a JavaCard applet that emulates the HMAC challenge-response functionality of the Yubikey NEO/4/5. It presents the same interface that a real Yubikey presents over CCID (i.e. this applet does not have any HID features).

The goal is to be able to write applications that use the HMAC-SHA1 Challenge-Response mode of the Yubikey, and have a JavaCard with this applet be a drop-in replacement.

### Current status

What works:

 * HMAC-SHA1 challenge response, in HMAC_LT64 mode
 * Setting configuration using `CMD_SET_CONF_{1,2}`
 * Using the protection access code to prevent accidental slot overwrite

## Building

All of the procedures have been tested on Linux, but should work on Windows and OSX as well. The following instructions are for Linux. 

It is also recommended to use VS Code, because then you can use the configured tasks instead of typing the all the commands out.

Clone this repository:

```
$ git clone --recurse-submodules https://github.com/StarGate01/vk-ykhmac
```

Make sure you are using JDK 8. Install [Ant](https://ant.apache.org/) (might be available in your package manager) Then compile that applet like this, you can also use the configured VS Code build task.

```
$ ant dist
```

## Deployment

Upload the `target/YkHMACApplet.cap` file to your card using [fdsm](https://github.com/fidesmo/fdsm):

```
$ java -jar fdsm.jar --install target/YkHMACApplet.cap
```

You might have to compile `fdsm` yourself, and even switch to a more recent JDK for it.

Since you probably do not have the application keys to deploy this specific AID, you can adjust the IDs in `build.xml` to match your Fidesmo developer keys.

## APDU Testing

Install [Maven](https://maven.apache.org/) (might be available in your package manager), then build  the **jcardsim** submodule or use the configured VS Code build task:

```
$ cd tools/jcardsim
$ JC_CLASSIC_HOME=../../sdks/jc305u3_kit/ mvn initialize
§ JC_CLASSIC_HOME=../../sdks/jc305u3_kit/ mvn clean install
```

Then test the applet using some sample APDUs, or use the configured VS Code test task:

```
$ java -cp tools/jcardsim/target/jcardsim-3.0.5-SNAPSHOT.jar:./target com.licel.jcardsim.utils.APDUScriptTool test/jcardsim.cfg test/apdu.script
```

## PC/SC Emulation

Install [vsmartcard](https://frankmorgner.github.io/vsmartcard/) (might be available in your package manager) and make sure it runs and connects to your PC/SC service.


Then emulate the applet, or use the configured VS Code test task:

```
$ java -cp tools/jcardsim/target/jcardsim-3.0.5-SNAPSHOT.jar:./target com.licel.jcardsim.remote.VSmartCard test/jcardsim.cfg
```

If everything works, a reader `Virtual PCD 00 00` with the applet emulated in it should show: 

<details>
<summary>pcsc_scan</summary>

```
$ pcsc_scan

 Reader 0: Virtual PCD 00 00
  Event number: 13
  Card state: Card inserted, 
  ATR: 3B 8D 80 01 80 73 C0 21 C0 57 59 75 62 69 4B 65 79 F9

ATR: 3B 8D 80 01 80 73 C0 21 C0 57 59 75 62 69 4B 65 79 F9
+ TS = 3B --> Direct Convention
+ T0 = 8D, Y(1): 1000, K: 13 (historical bytes)
  TD(1) = 80 --> Y(i+1) = 1000, Protocol T = 0 
-----
  TD(2) = 01 --> Y(i+1) = 0000, Protocol T = 1 
-----
+ Historical bytes: 80 73 C0 21 C0 57 59 75 62 69 4B 65 79
  Category indicator byte: 80 (compact TLV data object)
    Tag: 7, len: 3 (card capabilities)
      Selection methods: C0
        - DF selection by full DF name
        - DF selection by partial DF name
      Data coding byte: 21
        - Behaviour of write functions: proprietary
        - Value 'FF' for the first byte of BER-TLV tag fields: invalid
        - Data unit in quartets: 2
      Command chaining, length fields and logical channels: C0
        - Command chaining
        - Extended Lc and Le fields
        - Logical channel number assignment: No logical channel
        - Maximum number of logical channels: 1
    Tag: 5, len: 7 (card issuer's data)
      Card issuer data: 59 75 62 69 4B 65 79
+ TCK = F9 (correct checksum)

Possibly identified card (using /usr/share/pcsc/smartcard_list.txt):
3B 8D 80 01 80 73 C0 21 C0 57 59 75 62 69 4B 65 79 F9
	Yubikey 5 NFC (via NFC) (Other)
	https://www.yubico.com/product/yubikey-5-nfc/#yubikey-5-nfc
```

</details>

<details>
<summary>opensc-tool</summary>

```
$ opensc-tool -l

# Detected readers (pcsc)
Nr.  Card  Features  Name
0    Yes             Virtual PCD 00 00
1    No              Virtual PCD 00 01

```

</details>

Next, send the initial boot APDU to create the applet, or use the VS Code  test task configured:

```
$ opensc-tool -r 'Virtual PCD 00 00' -s '80 b8 00 00 0e 07 a0 00 00 05 27 20 01 05 00 00 02 F F 7f'
Sending: 80 B8 00 00 0E 07 A0 00 00 05 27 20 01 05 00 00 02 FF 7F 
Received (SW1=0x90, SW2=0x00):
A0 00 00 05 27 20 01 ....' .
```

From there on, the applet should behave like a hardware card running the applet and should be able to accept and generate HMACs.

## Programming a secret

Build the **yktool** submodule, or use the configured VS Code build task:

```
$ cd tools/yktool
$ make yktool.jar
```

Then use it to program a secret:

```
$ cd tools/yktool
$ java -jar yktool.jar list
Yubikeys available:
  - Yubikey 4 #279305487 v4.0.0

$ echo 'b6e3f555562c894b7af13b1db37f28deff3ea89b' | java -jar yktool.jar program hmac 1 -x -X
Programmed slot 1 ok

$ printf 'aaaa' | java -jar yktool.jar hmac 1 -x
72:7E:C8:E8:15:EE:C5:32:8F:9D:9C:BE:5E:F2:4E:A8:36:D7:CE:56
```

## Thanks to

- https://github.com/arekinath/YkOtpApplet
- https://github.com/arekinath/PivApplet#simulation
- https://github.com/arekinath/yktool
- https://github.com/arekinath/jcardsim
- https://github.com/martinpaljak/oracle_javacard_sdks