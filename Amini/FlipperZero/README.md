## Changing Flipper Zero advertising BLE information

The file modified was  *gap.c*. In this example, the Flipper device will mimic the Apple AirTag advertising behavior.

The main modifications were in/with the functions:
- aci_gap_set_discoverable
- aci_gap_delete_ad_type
- aci_gap_update_adv_data
```
...
static void gap_advertise_start(GapState new_state) {
    FURI_LOG_E(TAG, "gap_advertise_start");

    tBleStatus status;
    uint16_t min_interval;
    uint16_t max_interval;

    if(new_state == GapStateAdvFast) {
        min_interval = 0x80; // 80 ms
        max_interval = 0xa0; // 100 ms
    } else {
        min_interval = 0x0640; // 1 s
        max_interval = 0x0fa0; // 2.5 s
    }
    // Stop advertising timer
    furi_timer_stop(gap->advertise_timer);

    if((new_state == GapStateAdvLowPower) &&
       ((gap->state == GapStateAdvFast) || (gap->state == GapStateAdvLowPower))) {
        // Stop advertising
        status = aci_gap_set_non_discoverable();
        if(status) {
            FURI_LOG_E(TAG, "set_non_discoverable failed %d", status);
        } else {
            FURI_LOG_D(TAG, "set_non_discoverable success");
        }
    }

    static const uint16_t gap_appearance = 0x0000; //GAP_APPEARANCE_UNKNOWN

    status = aci_gatt_update_char_value(gap->service.gap_svc_handle,
                                gap->service.gap_svc_handle,
                                0,
                                sizeof(gap_appearance),
                                (uint8_t *)&gap_appearance);

    // Configure advertising
    status = aci_gap_set_discoverable(
        ADV_IND,
        min_interval,
        max_interval,
        PUBLIC_ADDR,
        0,
        0, NULL,                                  // Do not use a local name
        0, NULL,                                  // Do not include the service UUID list
        0,
        0);

    status = aci_gap_delete_ad_type(AD_TYPE_FLAGS); //Remove AD to free advertising buffer
    status = aci_gap_delete_ad_type(AD_TYPE_TX_POWER_LEVEL); //Remove AD to free advertising buffer
    
    const uint8_t airtag_adv[] = { //Data to be advertise simulating AirTag package
      0x1E, 0xFF, 0x4C, 0x00, 0x12, 0x19, 0x10, 0x12, 0x12, 0x34, 0x56, 
      0x78, 0x12, 0x34, 0x56, 0x78, 0x90, 0x12, 0x34, 0x56, 0x78, 0x90,
      0x12, 0x34, 0x56, 0x12, 0x34, 0x56, 0x12, 0x45
      }; 

    status = aci_gap_update_adv_data(0x1F, airtag_adv); //Update advertising data

    gap->state = new_state;
    GapEvent event = {.type = GapEventTypeStartAdvertising};
    gap->on_event_cb(event, gap->context);
    furi_timer_start(gap->advertise_timer, INITIAL_ADV_TIMEOUT);

}
...
```
Partial example of advertising [Twitter post](https://twitter.com/Netxing/status/1584654410856423424).
