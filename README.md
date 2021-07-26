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

## Emulation

Install [Maven](https://maven.apache.org/) (might be available in your package manager), then build  the **jcardsim** submodule or use the configured VS Code build task:

```
$ cd tools/jcardsim
$ JC_CLASSIC_HOME=../../sdks/jc305u3_kit/ mvn initialize
§ JC_CLASSIC_HOME=../../sdks/jc305u3_kit/ mvn clean install
```

Then test the applet using some sample APDUs, or use the configured VC Code test task:

```
$ java -cp tools/jcardsim/target/jcardsim-3.0.5-SNAPSHOT.jar:./target com.licel.jcardsim.utils.APDUScriptTool test/jcardsim.cfg test/apdu.script
```

**PC/SC Emulation is a work in progress and does not yet work**

Install [vsmartcard](https://frankmorgner.github.io/vsmartcard/) and make sure it runs and connects to your PC/SC service. 

Then emulate the applet, or use the configured VS Code test task:

```
$ java -cp tools/jcardsim/target/jcardsim-3.0.5-SNAPSHOT.jar:./target com.licel.jcardsim.remote.VSmartCard test/jcardsim.cfg
```

If everything works, `pcsc_scan` should show a reader `Virtual PCD 00 00` with the applet emulated in it.

### Programming the secret

Build the **yktool** submodule, or use the configures VS Code build task:

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