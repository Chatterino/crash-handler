# crash-handler

This repository contains the crash handler used in Chatterino.
The crash handler uses [crashpad](https://github.com/getsentry/crashpad) to capture exception information and restarts the application when a crash is captured.

The following [annotations](https://chromium.googlesource.com/crashpad/crashpad/+/refs/heads/main/handler/crashpad_handler.md#options) are used:

- `exePath` (required): an absolute path to the client executable
- `canRestart` (required): a boolean whether the handler should restart the client (`true` or `false`)
- `startedAt` (required): a UTC timestamp in ISO 8601 when the client was started
- `exeArguments` (optional): an encoded argument string for the client executable. Spaces are encoded as '+' and a plus is escaped by another plus ('+' → '++').

In addition to the arguments specified in `exeArguments`, the handler will start the client with the following arguments:

- `--crash-recovery`
- `--cr-exception-code <code>`: the numeric, native, platform-specific code of the exception that occurred
- `--cr-exception-message <message>`: text version of the exception code (potentially with more context)

## Building

This project is built with CMake. Currently, only Windows is supported. Run the following commands in a prompt with Visual Studio environment variables.

```shell
git clone --recurse-submodules https://github.com/Chatterino/crash-handler
cd crash-handler
cmake -G Ninja -B build
ninja -C build
```

The executable can be found in `build/bin/crashpad/crashpad-handler.exe`.
