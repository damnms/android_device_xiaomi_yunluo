# Device tree for the Redmi Pad (yunluo)

## Notes

- **Wi-Fi vendor HAL:** the stock `libwifi-hal-mtk.so` predates the
  Android 14 QPR2 `wifi_hal_fn` layout. `BoardConfig.mk` therefore sets
  `mediatek_wifi_hal.use_pre_u_qpr2_struct`, so that
  `hardware/mediatek/wlan/libwifi-hal-wrapper` reads the blob's function
  table correctly. Without it `android.hardware.wifi-service` crashes in
  `wifi_multi_sta_set_primary_connection` and Wi-Fi stays off.
- **Wi-Fi country code:** the tablet has no SIM, so the framework takes
  the country code from the country information of nearby access points
  (`config_wifiUpdateCountryCodeFromScanResultGeneric` in
  `WifiResOverlay`). At least two access points with -85 dBm or better
  must report the same country. Until then no country is set and the
  DFS channels 52-144 are not scanned; channels 36-48 always work. The
  MediaTek driver ignores 802.11d itself
  (`REGULATORY_COUNTRY_IE_IGNORE`), so this cannot be done in the driver.
- **tinyxml2:** the platform `libtinyxml2` is no longer ABI-compatible
  with the Android 14 MediaTek blobs. `extract-files.py` relinks them
  against `libtinyxml2-v34` from `hardware/lineage/compat`.
- **Video decoder on the 3 GB model:** `mtk-vcu.ko` allocates a 123 KB
  queue with `kzalloc()` for every new decoder. After some hours of use
  physical memory is too fragmented for that, the decoder init fails
  ("Allocate new vcu queue fail!") and videos stop playing in all apps
  until a reboot. `device/xiaomi/yunluo-kernel` therefore carries a
  rebuilt `mtk-vcu.ko` that uses `kvzalloc()`; the patch is in
  `yunluo-kernel/patches/`.
