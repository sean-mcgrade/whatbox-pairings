# Lineage — android/sean-4.1-mod

Sean McGrade — 4.1-modernization branch.
Introduces IMachineDriver abstraction: BluetoothClassicDriver + TelnetDriver + BLEDriver.
Pluggable transport — targets firmware v4.1 + external ESP32 bridge (whatbox-espbluetooth) for Telnet.
Adds EEPROM_DATA 'O' packet. Drops X J H Q. ButterKnife migration, Android 12+ permissions.
Upstream: https://github.com/sean-mcgrade/whatbox-android-remote-control branch preserve/4.1-modernization @ e6f377c
