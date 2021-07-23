# VivoKey fork of YkOtpApplet

Parent repository: https://github.com/arekinath/YkOtpApplet by Alex Wilson
## About

This is a JavaCard applet that emulates the HMAC challenge-response
functionality of the Yubikey NEO/4. It presents the same interface that a real
Yubikey presents over CCID (i.e. this applet does not have any HID features).

The goal is to be able to write applications that use the HMAC-SHA1
Challenge-Response mode of the Yubikey, and have a JavaCard with this applet
be a drop-in replacement.

## Current status

What works:

 * HMAC-SHA1 challenge response, in HMAC_LT64 mode
 * Setting configuration using `CMD_SET_CONF_{1,2}`
 * Using the protection access code to prevent accidental slot overwrite

## Installing

The pre-built `.cap` files for each release can be found on the
https://github.com/arekinath/ykotpapplet/releases[project release page].

You can use the
https://github.com/martinpaljak/GlobalPlatformPro[Global Platform] command-line
tool (`gp`) to upload the applet to your JavaCard:

-----
$ gp -install YkOtpApplet.cap
CAP loaded
-----

The easiest way to program the applet with an HMAC secret is to use
https://github.com/arekinath/yktool[yktool]:

-----
$ yktool list
Yubikeys available:
  - Yubikey 4 #279305487 v4.0.0

$ echo 'b6e3f555562c894b7af13b1db37f28deff3ea89b' | yktool program hmac 1 -x -X
Programmed slot 1 ok

$ printf 'aaaa' | yktool hmac 1 -x
72:7E:C8:E8:15:EE:C5:32:8F:9D:9C:BE:5E:F2:4E:A8:36:D7:CE:56
-----

## Building the project

We use https://github.com/martinpaljak/ant-javacard[ant-javacard] for builds.

-----
$ git clone https://github.com/arekinath/YkOtpApplet
...

$ cd YkOtpApplet
$ git submodule init && git submodule update
...

$ export JC_HOME=/path/to/jckit-2.2.2
$ ant
-----

The capfile will be output in the `./bin` directory, along with the `.class`
files (which can be used with jCardSim).
