# GPU suspend-point lifecycle correction

The native submission adapter now calls `sceAgcSuspendPoint(void)` after a
successful `sceAgcDriverSubmitDcb`. The import NID is `h9z6+0hEydk`.
The declaration and link facade resolve to the console module, not an emulated
success implementation. The context requires both callbacks.

The motivating ps5vk FW12.02 control completed rendering and freed tracked
resources, but closing generated
`CPU_FAULT_SUSPENDPOINT_TIMEOUT_IN_SUSPEND_ASYNC` with `in_frame=1`.
After adding suspend points, the same diagnostic scene passed two close/relaunch
cycles; the first kernel trace showed suspension in 75ms and `in_frame=0`.
This is reference evidence from ps5vk, **not hardware validation of this repo**.

Host submission tests cover submit failure (no suspend call), successful ordering,
suspend failure propagation and missing callback rejection. A successful submit
followed by suspend failure still means submitted work: existing conservative
failure handling must retain resources. Neither return zero nor a suspend point
replaces exact GPU completion, VideoOut retirement or safe command-buffer reuse.

Native build and same-artifact hardware close/relaunch validation for this
repository remain pending until explicitly recorded. Historical render/soak
evidence does not automatically certify this changed binary.

Full `make test` passed. `xash/build_engine.sh` builds the AGC PRX from
`native/ps5_agc_native.c` and `src/ps5_agc_submit.c`, so the fix is in the
engine renderer's source path as well as the standalone backend. The installed
application has not been replaced; a new engine/PRX build and hardware test are
still needed. Test-target cleanup removed generated build outputs, not sources
or installed console content; normal build targets regenerate them.
