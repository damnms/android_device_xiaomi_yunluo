# Device tree for the Redmi Pad (yunluo)

## Notes

- **Wi-Fi vendor HAL:** the stock `libwifi-hal-mtk.so` predates the
  Android 14 QPR2 `wifi_hal_fn` layout. `BoardConfig.mk` therefore sets
  `mediatek_wifi_hal.use_pre_u_qpr2_struct`, so that
  `hardware/mediatek/wlan/libwifi-hal-wrapper` reads the blob's function
  table correctly. Without it `android.hardware.wifi-service` crashes in
  `wifi_multi_sta_set_primary_connection` and Wi-Fi stays off.
