# HSS1394

*High-speed MIDI-over-Firewire device access library for Windows and macOS.*

HSS1394 is a high-speed MIDI-over-Firewire protocol developed by Stanton Magnetics, Inc.
This project contains the library code that interfaces with HSS1394 devices on Windows and macOS, such as the Stanton SCS.1 controller series.
This library is unnecessary on Linux because support for these devices is integrated into the kernel (ALSA).

This library is maintained by the [Mixxx DJ Software](https://mixxx.org/) team.
Contributions are welcome by opening pull requests and issues on [GitHub](https://github.com/mixxxdj/hss1394).
If you want to discuss anything about this library with us, please get in touch on our [Zulip chat](https://mixxx.zulipchat.com/).

## Building

HSS1394 uses the [CMake build system](https://cmake.org/).
From the top directory of this source code repository, run:

    $ cmake -DCMAKE_INSTALL_PREFIX=/path/to/install/to -S . -B build
    $ cmake --build build
    $ cmake --install build

## License

The HSS1394 library is licensed under the terms of the GNU Lesser General Public License (LGPL) version 3 or later.
See [`LICENSE`](LICENSE) for details.
