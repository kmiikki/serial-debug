# serial-debug

![Platform](https://img.shields.io/badge/platform-Linux-blue)
![Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)

`serial-debug` is a Bash tool for debugging serial communication with
laboratory instruments, sensors, embedded devices, and industrial
equipment.

It works as a low-level serial protocol debugger by showing all traffic
in both hexadecimal and ASCII form. This makes it possible to determine:

- correct baud rate
- correct line ending (`CR`, `LF`, `CRLF`, ...)
- actual bytes sent and received
- whether a device is responding at all

Unlike programs such as `minicom`, `screen`, or `picocom`, this tool is
designed for situations where the correct communication settings are not
known in advance.

Supports:

- RS-232
- RS-485
- UART

The tool is implemented as a single portable Bash script with no external
dependencies.

## Features

- hex + ASCII traffic display
- optional decoded text line
- automatic baud-rate scanning
- automatic EOL scanning
- address scanning
- interactive command mode
- listen/sniff mode
- millisecond or microsecond timestamps
- logging to file
- optional prefix bytes before commands
- suitable for RS-232, RS-485, and UART devices
- single-file Bash implementation with no dependencies

<p align="center">
<img src="images/scan-baud.png" width="900">
</p>

---

## Typical use cases

`serial-debug` is especially useful when:

- the baud rate is unknown
- the required line ending is unknown
- an instrument does not respond in a normal terminal program
- an RS-485 device requires a prefix or address
- a device returns binary or partly non-printable data
- communication must be logged for later analysis

Typical examples include:

- DPS8000 pressure sensors
- laboratory balances
- industrial RS-485 instruments
- custom microcontroller projects
- USB-UART adapters

---

## Example: EOL scanning

Many instruments require a specific line ending (`CR`, `LF`, `CRLF`, or `LFCR`).
When documentation is unclear, `serial-debug` can test them automatically.

```bash
./serial-debug --scan-eol /dev/ttyUSB0 I4
````

![EOL scan](images/scan-eol.png)

The correct line ending becomes obvious as soon as the device responds.

---

## Example: Baud-rate scanning

Another common problem is unknown baud rate.

```bash
./serial-debug --scan-baud /dev/ttyUSB0 S
```

The correct baud rate becomes immediately visible when a response appears.

---

## Example: DPS8000 direct mode

DPS8000 sensors typically require:

* a leading space character before the command
* `CR` line ending

Example:

```bash
./serial-debug -p sp -e cr /dev/ttyUSB0 "1:N,?"
```

This sends:

```text
<SP>1:N,?<CR>
```

which is required by many DPS8000 sensors to stop autoread mode and
execute a direct command.

---

## Example: Address scanning

To find which RS-485 addresses respond:

```bash
./serial-debug --scan-addr -p sp -e cr /dev/ttyUSB0 "N,?"
```

Example output:

```text
Trying address 1 (1/32): 1:N,?
Trying address 2 (2/32): 2:N,?
...
Address 4 responded
```

This is useful when device addresses are unknown or when recovering a
misconfigured RS-485 bus.

---

## Quick start

Clone the repository:

```bash
git clone https://github.com/kmiikki/serial-debug.git
cd serial-debug
```

Make the script executable:

```bash
chmod +x serial-debug
```

Basic usage:

```bash
./serial-debug /dev/ttyUSB0 S
```

Interactive mode:

```bash
./serial-debug --interactive /dev/ttyUSB0
```

Listen mode:

```bash
./serial-debug --listen /dev/ttyUSB0
```

---

## Usage

```text
Usage:
  serial-debug [options] DEVICE COMMAND
  serial-debug [options] --interactive DEVICE
  serial-debug [options] --listen DEVICE
  serial-debug [options] --sniff DEVICE
  serial-debug [options] --scan-eol DEVICE COMMAND
  serial-debug [options] --scan-baud DEVICE COMMAND
  serial-debug [options] --scan-addr DEVICE COMMAND

General:
  -b, --baud RATE        Set baud rate (default: 9600)
  -e, --eol MODE         Set line ending:
                         none | cr | lf | crlf | lfcr
  -p, --prefix MODE      Prefix command with:
                         none | sp | tab | esc
  -w, --wait SEC         Wait time after command (default: 0.5)

Modes:
  --scan-eol             Test common line endings
  --scan-baud            Test common baud rates
  --scan-addr            Test addresses 1-32
  --interactive          Interactive command mode
  --listen               Listen continuously
  --sniff                Alias for --listen

Output:
  --text                 Show decoded text line
  --ts-ms                Show millisecond timestamps
  --ts-us                Show microsecond timestamps
  --width N              Set output width

Logging:
  --log FILE             Write output to log file
  --append               Append to existing log

Other:
  -h, --help             Show help
  --version              Show version
```

---

## DPS8000 notes

For DPS8000 sensors, the following settings are usually correct:

```bash
-p sp -e cr
```

Examples:

```bash
./serial-debug -p sp -e cr /dev/ttyUSB0 "N,?"
./serial-debug -p sp -e cr /dev/ttyUSB0 "1:U,?"
./serial-debug -p sp -e cr /dev/ttyUSB0 "1:P,?"
```

Many DPS8000 commands require the direct-mode stop prefix (`SP`) before
the actual command.

---

## Output example

```text
TX 00000000  20 31 3A 4E 2C 3F 0D                          | 1:N,?.
RX 00000000  30 31 0D                                      |01.
```

With `--text` enabled:

```text
TEXT: 01
```

---

## Examples

Additional usage examples can be found in:

```text
examples/usage-examples.txt
examples/dps-dump.txt
```

---

## Why this tool exists

Many laboratory instruments and industrial devices still use serial
communication, but debugging them can be unnecessarily difficult.

Most terminal programs assume that the correct settings are already known.

In practice this is often not true.

`serial-debug` was created to make unknown serial communication visible
and easy to understand.

It helps determine:

* correct baud rate
* correct line ending
* required prefixes or address format
* actual byte-level communication

quickly and reliably.

---

## Tested with

* DPS8000 pressure sensors
* laboratory balances
* RS-485 sensors
* embedded devices
* USB serial adapters

---

## Platform

Linux (tested)

The tool uses standard Linux serial device interfaces such as:

```text
/dev/ttyUSB*
/dev/ttyACM*
```

Other Unix-like systems (for example macOS) may work but are currently
untested.

---

## License

This project is licensed under the MIT License.

Copyright (c) 2026 Kim Miikki
