# runner-background-cancel-repro

Minimal repro: on Windows, `cancel:` on one background step also interrupts
every other step that is running at that moment.

The workflow starts two background `node` steps, cancels one (`target`), and
waits for the other (`bystander`), which should finish on its own after 20 s.
It runs five times each on `windows-latest` and `ubuntu-latest`.

- **ubuntu-latest:** the bystander finishes; the leg passes.
- **windows-latest:** the bystander logs `bystander got SIGINT` at the moment
  of the cancel and fails.

Likely cause, from the runner source: `ScriptHandler` starts steps with
`inheritConsoleHandler: true`, so every step shares the worker's console, and
`ProcessInvoker.SendCtrlSignal` does `AttachConsole(step)` then
`GenerateConsoleCtrlEvent(sig, 0)`, where group 0 means every process on
that console.
