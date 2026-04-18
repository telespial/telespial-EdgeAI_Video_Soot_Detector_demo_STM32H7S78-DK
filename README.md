# Soot Detector

Soot Detector is an STMicroelectronics-focused vision demo for contamination awareness on `STM32H7S78-DK`.

Primary objective for this phase:
- Preserve and iterate from the current ST demo behavior (Life.Augmented splash, then `Future Electronics` text freeze state).

## Target Hardware
- Vendor: STMicroelectronics
- Dev kit: STM32H7S78-DK
- MCU family: STM32H7RS (Cortex-M7)

## Firmware Safety
- Current board firmware backup is stored in:
  - `firmware_backups/stm32h7s78dk_flash_backup_*.bin`
  - `firmware_backups/stm32h7s78dk_flash_backup_*.meta.txt`
- Use this backup as restore point before replacing firmware images.

## Runtime Pipeline
1. Live camera frame input.
2. Vision adapter extracts numeric degradation features.
3. EIL evaluates features with anomaly/adaptive model.
4. UI/alerts present contamination state.

## Quick Workflow
1. Open this project from EmbeddedX.
2. Open EIL Model Config Editor for this project.
3. Start live capture.
4. Record clean baseline first.
5. Run replay/compare diagnostics.
6. Review output artifacts in `outputs/`.

## Onboarding Docs
- Entry: `start_here.md`
- Canonical read order: `docs/START_HERE.md`

## Baseline Guidance
- Recommended baseline duration: 120 seconds.
- Recommended minimum samples: 200.
- Keep camera framing and lighting stable during baseline capture.

## USB Webcam Bug Status (2026-03-19)
Current symptom:
- USB1 VBUS is stable and camera powers on, but webcam LED remains red and no live video stream is visible.

What is confirmed:
- USB1 is the active host path in this firmware branch.
- USB2 is not the active webcam-host path for current bring-up.
- Touch/UI remains operational while USB host-camera experiments run.

What has already been attempted:
- Forced USB1 host-only mode (disabled DRP device switching behavior in app flow).
- Kept USB1 VBUS source asserted to avoid 500 ms power drop-off.
- Reworked UVC negotiation and streaming sequence in `usb_host.c`:
  - `GET_PROBE -> SET_PROBE -> SET_COMMIT -> SET_INTERFACE`
  - Probe-length fallback handling (`34` and `26`)
  - Alternate-setting retries
  - Iso endpoint sizing fixes from `wMaxPacketSize` fields
- Rebuilt/flashed repeatedly with verify after each change set.

Current blocker:
- Webcam still does not transition from powered-idle (red LED) to active-stream state.

Immediate next step:
- Continue UVC host debug in `Appli/Src/usb_host.c` with explicit stage diagnostics and descriptor-driven handshake validation.

## Display Issue Summary (2026-03-17)
Symptom:
- Screen showed repeated/tiled copies of the UI/background (appeared as ~5 repeated images), not a single full-screen frame.

Root cause:
- TouchGFX runtime geometry was set to landscape (`800x480`) while the generated screens and background assets are portrait (`480x800`).
- This mismatch caused framebuffer/stride interpretation errors that presented as repeated tiles.

Fix applied:
- Runtime geometry was aligned to portrait in:
  - `TouchGFX/target/generated/src/TouchGFXConfiguration.cpp`
  - `TouchGFXHAL hal(..., 480, 800);`
  - `nema_vg_init_stencil_pool(480, 800, 1);`
- Firmware was rebuilt and reflashed with verify.

Important note:
- The background image file itself was not the primary fault once this mismatch was present.
- If the repeat pattern returns, check geometry first before changing PNG assets.

## Pre-Flash Geometry Checklist
1. Confirm generated screen sizes are `480x800`.
2. Confirm `TouchGFXHAL` width/height are `480x800`.
3. Confirm Nema stencil pool is `480x800`.
4. Rebuild from clean state.
5. Flash with verify and reset.

## Folder Map
- `captures/`: clean/dirty/raw/synthetic captures.
- `features/`: extracted feature streams + schema.
- `models/`: EIL config/template files.
- `vision_adapter/`: adapter settings and scripts.
- `outputs/`: compare/replay/reports artifacts.
- `mrd/`: project-local hardware descriptors.
- `firmware_backups/`: binary restore points from board flash.

## License

See [LICENSE](./LICENSE).
