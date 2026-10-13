# Dyson (patched) — BLE lights over ESPHome Bluetooth proxies

Patched fork of [cmgrayb/hass-dyson](https://github.com/cmgrayb/hass-dyson) (MIT, see `LICENSE`),
published only so HACS can install it on one Home Assistant instance that controls
Dyson Solarcycle/Lightcycle Morph lamps (product family `E5R`) through an ESPHome
Bluetooth proxy.

Two behaviour changes versus upstream, both verified against real lamps:

1. **Write type follows the characteristic's advertised properties.** The Morph
   family advertises only `write-without-response` on 11000/11001/11009/0021, so an
   acknowledged ATT Write Request is refused with `0x03 Write Not Permitted`.
   `_wants_write_request()` chooses the flag per characteristic instead of assuming
   the MyDyson app's behaviour. Lamps that do advertise `write` are unaffected.
2. **CCCD is written explicitly after `start_notify`.** An ESPHome Bluetooth proxy
   accepts the subscription but leaves the client characteristic configuration
   descriptor at `0000`, so the lamp never notifies and the LTK handshake times out.
   `_enable_cccd()` writes `0x0001`; on a direct adapter this is a no-op.

Everything else is upstream's code and test suite (3 165 tests pass).
