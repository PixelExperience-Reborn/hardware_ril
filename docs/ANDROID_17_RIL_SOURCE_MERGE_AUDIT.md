# PixelExperience-Reborn Android 17 — hardware/ril source port audit

**2026-11-05 | P2 matrix row 154 | Official AOSP17 source merge complete; device integration OPEN.**

## Source ancestry and reviewed conflict

- Historical PE `seventeen`: `b34bd171db21c0bca7ae0c867d224a29b4e731a7`, with PE patch wrapping the old Make `libril/Android.mk` in `ifneq ($(BOARD_PROVIDES_LIBRIL),true)`.
- Exact official AOSP17 source: `platform/hardware/ril` tag `android-17.0.0_r1`, commit `35d736e2410f8b49e3b6f3a5649dc9e9d24ccd3c`, Git tree `fd290e7a4c11c04092a10c887aa4c5ddcc6fa470`.
- Source history was merged on the isolated `integrate/a17-aosp17-ril-20261105` branch, after requiring both exact commit and tree SHA plus shared Git ancestry.
- The one reviewed modify/delete conflict was `libril/Android.mk`: official AOSP17 removed this file and uses `libril/Android.bp`, `cc_library_shared { name: "libril", vendor: true }`. The old PE file depended on the Make-only `BOARD_PROVIDES_LIBRIL` guard. CI required this to be the **only** unresolved path, verified official Soong module ownership, and removed that obsolete Makefile. Historical PE commits remain available via Git ancestry; no fake `Android.mk` reintroduced or global conflict override.
- Exact source merge commit: `9e87c0a26a1b2a61147037f2d202c5c7ca8ac2ca`, [workflow #42007705698 SUCCESS](https://github.com/PixelExperience-Reborn/hardware_ril/actions/runs/42007705698). This was a **source merge/integrity** check, not an Android17 ROM build.

## Modern source contract and mondrian boundaries

- `libril/Android.bp` uses modern Soong `cc_library_shared`, vendor partition, HIDL `android.hardware.radio@1.0/@1.1` native interfaces and `libbinder`-independent legacy RIL code paths, plus Soong configuration for `ril.sim_count` and `ril.disable_rild_oem_hook`.
- `rild/Android.bp` defines proprietary `rild` **disabled unless** `soong_config_variable("ril", "use_aosp_rild")` is true. The AOSP17 RIL source module does **not** justify forcibly starting a generic AOSP `rild` on Qualcomm vendor radio.
- The current read-only `LF52406/mondrian_android_device_xiaomi_sm8450-common` (`lineage-24.0`) sets `ENABLE_VENDOR_RIL_SERVICE := true` in `BoardConfigCommon.mk` and selects vendor telephony extensions, `qcrilNrDb_vendor`, telephony support resources in `common.mk`; `BOARD_PROVIDES_LIBRIL` is **not** in the inspected `BoardConfigCommon.mk`. Full device-tree and vendor blob audit is still needed to establish the modem service and selected `libril` owner.
- Official Android17 `packages/modules/Telephony`, `packages/services/Telephony`, `frameworks/opt/telephony`, `hardware/interfaces` and vendor Radio AIDL/HIDL contracts are **separate** from this legacy C++ RIL library. Merging this source is not proof of modem compatibility, IMS or VoLTE.
- AOSP17 RIL interface modules, Soong namespace, `librilutils` radio and Bluetooth APEX compatibility must be validated. Do not port old board macros globally or invent `android.hardware.radio` service declarations.

## Required acceptance gates

1. Pin this audited source commit once in a **separate P2 manifest profile** replacing only the existing `hardware/ril` checkout; never activate it as a second conflicting source tree.
2. Validate source-integrity and official `repo manifest` project graph, including the old PE and official AOSP17 ancestry.
3. In the independently authored PE-Reborn mondrian product, resolve one vendor radio service and `libril` owner, then check full Soong compile of `libril`, `librilutils`, `rild` selection, `ril_headers`, and related radio interface libs.
4. Validate vendor binaries and linker symbols, vendor properties, SIM/UICC, RIL/IMS/telephony extensions, emergency call routing, carrier config, 5G, LTE+, APN/data, telephony audio, suspend/resume, SELinux enforcing and exact device VINTF.
5. Run Android17 `checkvintf`, CTS/VTS and physical mondrian modem tests. Do not claim working VoLTE icon or telephony until actually verified.

**Status:** reviewed exact Android17 Git source merge only; no complete Soong, VINTF, SELinux or physical modem testing. Keep the original `seventeen` and current MistOS/LineageOS device trees unchanged.

**Next P2 row:** `kernel/configs`.
