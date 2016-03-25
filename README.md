# Host Scanner

The purpose of this project is to discover hosts on a network and gather information about them for later analysis.

## Features

* TCP scanner
  * A high-performance TCP scanner which initiates the three-way handshake by multiplexing non-blocking sockets and grabs the service banner.

* UDP scanner
  * Uses a list of known port numbers and sends a specifically crafted payload in order to try and get an answer from the server, if there are any listening.

* ICMP pinger
  * Support for ICMP Echo Request packets (also known as "standard ping") to determine if a host is alive.

* ARP pinger
  * Support for ARP Request packets to map alive hosts on a local network.

* Host discovery
  * Discovers hosts behind a given netblock by sending ICMP Echo Request packets, failing that sends TCP SYN packets to most common open ports, failing that as well will result in sending specifically-crafted UDP packets to most common UDP-based services.

* External scanners
  * Ability to use external tools for all the scanning needs, instead of the built-in scanners. Currently Nmap support is implemented, more to follow if needed.

* Online sources
  * Support for passive reconnaissance by fetching already available data from relevant services intended for security researchers. Currently Shodan and Censys are supported.

* Unit tests
  * All features are covered by unit tests which are run on three platforms in order to ensure utmost stability and portability.

* Portability
  * Features are implemented (when a standardized API is not available) using raw sockets on Linux, WinPcap on Windows, and Berkeley Packet Filter on BSD / OS X.

### Planned features

* Network mapping
  * Neighbor Solicitation for IPv6

## Building

To compile and run the project, you must first install the dependencies, which can be done with:

 * Debian/Ubuntu/Kali:
 
        apt install build-essential cmake libcurl4-openssl-dev libsqlite3-dev libboost-all-dev libz-dev

 * RHEL/CentOS/Fedora:

        yum install gcc-c++ make cmake libcurl-devel sqlite-devel boost-devel-static zlib-devel

 * FreeBSD:

        pkg install cmake curl sqlite3 boost-libs

 * Mac OS X: (with [Homebrew](http://brew.sh/))

        xcode-select --install && brew install cmake curl sqlite boost

After the dependencies have been installed, you can check out the repository and compile it with the following commands:

    git clone https://github.com/RoliSoft/Host-Scanner.git
    cd Host-Scanner/build
    cmake ..
    make

If the compilation was successful, you can run it with the `./HostScanner` command. Tests are also available, you may run them through `make test` or directly, by executing `./TestScanner`.

## Distribution

The `distrib` folder contains `Dockerfile` scripts to pull, compile and package the latest revision. The `.deb` package is generated with the current Debian Stable (`debian:latest` in Docker Hub) while the `.rpm` package is generated with the current Fedora version. (`fedora:latest`)

Since Boost is statically linked during compilation, the only dependency for now is `libcurl` for both distributions.

The files will be generated by CMake's packaging component, CPack, whose configuration can be found in `CPackConfig.cmake`.

In order to set up these build environments, you must first compile the container:

    cd distrib/deb
    docker build -t debbuild .

After this, the `debbuild` container will be available for any compilations needs. To bake a fresh `.deb`, just run:

    docker run -it debbuild

This will pull a fresh copy of the repository _inside_ the container, therefore any changes to the repository on your host machine will not be reflected. If you wish to compile you own fork or a different branch, you'll have to modify the `compile.sh` file next to the preferred `Dockerfile` and rebuild the container.

## Portability

You'll need a fairly new compiler, as C++14 features are used in the code. As for platforms, the application is compiled and unit-tested periodically on the following:

 * Windows
   * Windows 10 / Visual Studio 2015

 * Linux
   * Debian stable, unstable / gcc, clang
   * Fedora latest, rawhide / gcc, clang

The project was developed with support in mind for the following platforms, however continuous integration is not available for these platforms, therefore builds might break until tested or otherwise observed:

 * BSD/Darwin
   * FreeBSD 11 / clang 3.7.0
   * OS X 10.11 / AppleClang 7.0.0

Other platforms are not supported at this time.

## Permissions

Some features of the application require elevated privileges in order to run:

* `IcmpPinger` uses raw sockets in order to send and receive ICMP packets.
  * *Windows*: Administrator privileges are required.
  * *Linux*: root user _or_ `CAP_NET_RAW` capability is required.
  * *BSD / OS X*: root user is required.

* `ArpPinger` crafts, sends and listens for raw Ethernet frames.
  * *Windows*: [WinPcap](http://www.winpcap.org/install/default.htm) driver is required.
  * *Linux*: root user _or_ `CAP_NET_RAW` capability is required.
  * *BSD / OS X*: root user _or_ read-write access to `/dev/bpf*` is required.

### Granting access

On Unix and Unix-like operating systems, if you wish to allow users without root privileges to run the application, you can do so by running:

    chmod +s HostScanner
    chown root:root HostScanner

This will activate the `SUID` bit, which will allow the application to escalate to root when run by an unprivileged user.

If you do not wish to run the application as root, but wish to use the features that require it, on Linux, you have the option of using the capabilities system:

    setcap cap_net_raw+eip HostScanner

This will specifically allow the use of raw sockets for this application when run by unprivileged users.

## Known issues

* Neither the `TcpScanner` nor the `UdpScanner` classes receive the `WSAECONNREFUSED` (`ECONNREFUSED` on Linux) error on Windows. There is little documentation on non-blocking sockets and this particular error. The [783052b](https://github.com/RoliSoft/Host-Scanner/commit/783052b49d39c3f2833e93c9bc183088eaec8797) commit tried using native `WSA*()` calls, as the documentation says `WSAECONNREFUSED` would be signalled on `FD_CONNECT`, but that is not happening, `WSAWaitForMultipleEvents()` returns either `WSA_WAIT_EVENT_0` or the undocumented `258` value, while `WSAGetLastError()` either returns `0` or `WSAEWOULDBLOCK`. The result of this issue is that on Windows, connections to non-listening ports will only be marked as dead after the timeout period has elapsed, while on Linux, the scan returns as soon as the ICMP packet is received.

## Licensing

Copyright (c) `2016` `RoliSoft <root@rolisoft.net>`

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version. This program is distributed in the hope that it will be useful, but without any warranty; without even the implied warranty of merchantability or fitness for a particular purpose.

For more information regarding the terms and conditions of this software, please read the full legal text of the GNU General Public License version 3, a copy of which is available in the [LICENSE.md](LICENSE.md) file. Otherwise, see &lt;<http://www.gnu.org/licenses/>&gt;.

Dual-licensing may be available upon request, depending on your purpose. For any inquiries, feel free to contact me at the email address listed above.