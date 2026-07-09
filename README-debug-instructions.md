# Debugging akonadi_maildir_resource Crash

## Overview
The `akonadi_maildir_resource` is crashing due to a Segmentation fault in `libKPim6AkonadiAgentBase.so` and `libKPim6AkonadiCore.so`, triggered by an asynchronous job completion signal (`KJob::result`).

Because debug symbols are currently stripped, the stack trace only shows `???` for the critical KDE PIM functions.

## Possible Causes based on Code Analysis

An initial review of the codebase (specifically `kdepim-runtime/resources/maildir/` and `akonadi/src/agentbase/resourcebase.cpp`) reveals that `ResourceBase` connects to the `KJob::result` signal in roughly 16 different ways to handle operations such as:
* Item/Folder Syncing (`slotItemSyncDone`, `slotCollectionSyncDone`)
* Saving changes (`changeCommittedResult`)
* Fetching item details (`slotItemRetrievalCollectionFetchDone`)
* Delivery of items (`slotDeliveryDone`)

The crash is highly likely a **use-after-free** or **null pointer dereference** happening in one of these slots when `AkonadiCore` attempts to process the payload or target collection of the job.

## Next Steps: Gathering a Full Backtrace

To accurately pinpoint and fix this crash, you will need to re-run the application with debug symbols installed so we can trace the exact function and line numbers.

### Instructions

1. **Install Debug Symbols:**
   Depending on your distribution, install the debuginfo/dbg packages for:
   * `kdepim-runtime` (or `kdepim-runtime-debuginfo`)
   * `akonadi` (or `akonadi-debuginfo`)
   * `kcoreaddons`
   * `qt6-qtcore` and `qt6-qtdbus`

   *Alternatively, if you are building from source, ensure you configure with `-DCMAKE_BUILD_TYPE=Debug`.*

2. **Reproduce the Crash:**
   Restart Akonadi and/or KMail and reproduce the crash.

3. **Capture the Backtrace:**
   Extract the crash backtrace using `coredumpctl`, `gdb`, or the KDE Crash Handler (DrKonqi).

4. **Provide the Trace to the Agent:**
   In your new session, provide the full backtrace with the debug symbols resolved.

### Prompt to use in your new session

You can use the following prompt to resume the investigation in your next session:

```text
Here is the backtrace for the akonadi_maildir_resource crash, this time with debug symbols enabled. Please analyze this trace, identify which KJob result slot in AkonadiAgentBase/AkonadiCore is causing the segmentation fault, and provide a patch to fix the null pointer or use-after-free bug.

[PASTE FULL BACKTRACE HERE]
```
