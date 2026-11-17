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
- **USB audio:** the stock `audio.primary.mediatek.so` (Android 14)
  reads USB audio devices through `libalsautils`. Since Android 14 QPR3
  (AOSP `79e3f23d`), `alsa_device_profile` has an extra field
  (`extra_latency_ms`) after `direction`, so the blob and the platform
  library disagree on the struct layout: the profile reads as invalid
  and USB headsets stay silent. `libalsautils-v34/` builds the
  `android14-release` version of `system/media/alsa_utils`, and
  `extract-files.py` links the audio HAL against it.
- **Dark video from 1080p on:** apps that record through a GPU surface
  (CameraX, e.g. Aperture) do not mark their stream as a video encoder
  consumer. The MediaTek HAL then treats a stream larger than the binned
  preview mode (1632x1224) as a still capture and picks the capture
  sensor mode, in which auto exposure stays at ISO 231 while streaming.
  `vendor.prop` sets `vendor.debug.cameng.force_sensormode=2`, so every
  session uses the normal video mode. In the `gc08a3` driver it has the
  same sensor setting as the capture mode (3264x2448, 30 fps), so photos
  keep their full resolution. `vendor_init` needs
  `set_prop` on `vendor_mtk_emcamera_prop` to set it at boot
  (`sepolicy/vendor/vendor_init.te`); without the rule the property stays
  empty.
- **OTA updates:** `system_ext.prop` sets `lineage.updater.uri` to
  `yunluo.json` in the main branch of
  [android_releases_yunluo](https://github.com/damnms/android_releases_yunluo).
  The LineageOS Updater reads that list (API v2 format) and downloads the
  zip from the GitHub release. Each release adds an entry with the
  build's `ro.build.date.utc` as `datetime`, `type` `unofficial`, and the
  zip's name, size, sha256 and download URL.
