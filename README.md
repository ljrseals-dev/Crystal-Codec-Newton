# Crystal ASIO Codec

**Crystal Codec — Crystal Cheairs and Rev. Cheairs — for Crystal City**  
**Crystal Codec - Newton** | Crystal City | Developed by Lewis J. Seals Jr. (Therealla) - ljrseals-dev  
**Version:** 1.0.0 core  
**Owner:** UnderJive-Records-Turnkey  
**License:** MIT

This repository is a from-scratch implementation of the portable core contracts
for the Crystal ASIO Codec. It is deliberately honest about platform
integration: the current build provides tested building blocks, not a
registered Windows ASIO driver or a verified WASAPI duplex backend.

## Implemented in this source tree

- Frame-based SPSC ring buffer with power-of-two capacity, monotonic counters,
  split wrap copies, cache-line-separated indices, and an explicit
  producer/consumer desync flag.
- PCM16/24/32 and float32 conversion routines with strict-aliasing-safe
  loads/stores and non-finite clamping.
- Config validation and local config read/write.
- Atomic counters for underruns, overruns, buffer switches, frames, and
  desyncs.
- Defensive Opus adapter: creation fails explicitly when libopus is absent;
  encode/decode reject undersized buffers, negative sizes, oversized packets,
  and null-packet-with-nonzero-length before libopus is invoked.
- Denormal guard (FTZ/DAZ on x86, FPCR.FZ on ARM64) for the audio thread.
- CMake build and unit tests.
- Compile-time local-only/no-network/no-telemetry/no-update-check definitions.

## Not implemented or not yet verified

- Steinberg ASIO 2.3 driver entry points and host buffer-switch integration.
- Windows WASAPI capture/render backend, event-driven COM lifecycle, MMCSS
  setup/teardown, and device reconnect.
- Worker thread orchestration and codec frame queueing.
- Drift compensation via IAudioClock.
- GUI/device enumeration, macOS CoreAudio, Linux JACK/ALSA.
- Physical loopback latency, real DAW validation, multi-hour stability, or
  24-hour test.

These are material gaps against the full master brief. Do not label this
package production-ready until those components are implemented and the
acceptance criteria are demonstrated on hardware.

## Build

```sh
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTS=ON
cmake --build build --parallel
ctest --test-dir build --output-on-failure
```

Optional libopus:

```sh
cmake -S . -B build -DCRYSTAL_ENABLE_OPUS=ON -DBUILD_TESTS=ON
```

Optional AVX2 is off by default because it must be enabled only for a
deployment fleet known to support it. Optional ASIO integration is gated
because the Steinberg SDK is separately licensed/distributed; this core does
not ship an invented or stubbed ASIO implementation.

## Realtime contract

Ring operations are intended for exactly one producer and one consumer.
Construct and allocate before streaming. `clear_quiescent()` may only be
called after producer and consumer stop. The API reports partial writes/reads;
callers must count underruns/overruns and define their silence/drop policy.
Do not call config or telemetry JSON functions from the real-time callback.

Construct one `RtDenormalGuard` per audio callback. Do not construct one per
inner loop iteration.

## Security

The core makes no network requests and contains no update checker or telemetry
exporter. The `CRYSTAL_NO_*` macros are build-policy markers, not a substitute
for auditing dependencies and binaries. MIT licensing applies to this project
code; libopus and any Steinberg SDK retain their own licensing terms.
