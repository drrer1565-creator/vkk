# Portable Reels export startup recovery

The released Windows portable r2 ZIP omitted the entire embedded Python runtime.
Its actual archive contains FFmpeg and FFprobe, but no python.exe. The production
isolated-renderer ready handler awaits resources/export-cpu-budget.js, which
starts resources/vendor/python/python.exe before dispatching the render command.
This produces ENOENT before any video frames are rendered.

The small native utility ACK smoke bypasses this production main-window handler,
so its successful 30-frame encode did not detect the incomplete application package.

Changes:

- Prepare the pinned upstream Python 3.11.9 embedded runtime, including its standard
  library and ctypes. Verify the official archive SHA-256 before extraction.
- An electron-builder afterPack hook rejects incomplete Windows runtime layouts
  and actually imports ctypes using the packaged python.exe on Windows.
- Preserve CPU budgeting; missing runtime reports an actionable error instead of
  requiring the user to install system Python.
- Await the isolated export promise before releasing its resource reservation.
  Cancel queued/active work promptly, remove listeners, and dispose the correct
  utility owner.
- Catch export preparation failures and show a native error dialog. Restore
  status, resume state and the export button even when preparation fails.
- Test the production main, preload, isolated renderer, utility and actual ASAR;
  verify frame counts, concurrent queueing, both cancellation paths, utility
  restart and the error-dialog IPC.
- Automatic verification no longer publishes or overwrites a GitHub Release.
  Publication requires a separate explicit workflow-dispatch opt-in.

Local validation: 50 Node tests, build and real FFmpeg media checks passed. The
production source and ASAR integration both encoded four 1080x1920 / 30fps /
21-frame jobs, cancelled queued and active exports, recovered after utility
restart and observed one native error dialog. Ongoing frame-control RPCs were
zero; final active sessions and encoder/preparation processes were zero.

Windows validation must additionally run the same integration against the built
win-unpacked/resources/app.asar with process.resourcesPath pointing to that exact
resources directory. This exercises the embedded CPU helper rather than a mock.
An artifact upload is delivery-only; inability to upload does not change any test
outcome. The user's real long-form projects and actual mouse dragging remain
separate acceptance checks.
