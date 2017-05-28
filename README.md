# XMR-Stak-CPU - Monero mining software

XMR-Stak is a universal Stratum pool miner. This is the CPU-mining version; there is also an [AMD GPU version](https://github.com/fireice-uk/xmr-stak-amd) and an [NVIDA GPU version](https://github.com/fireice-uk/xmr-stak-nvidia)

## Usage on Windows 
1) Edit the config.txt file to enter your pool login and password. 
2) Double click the exe file. 

XMR-Stak should compile on any C++11 compliant compiler. Windows compiler is assumed to be MSVC 2015 CE. MSVC build environment is not vendored.


### To do a static build for a system without gcc 5.1+
```
    cmake -DCMAKE_LINK_STATIC=ON .
    make install
```
Note - cmake caches variables, so if you want to do a dynamic build later you need to specify '-DCMAKE_BUILD_TYPE=RELEASE'


You can find a complete compile guide under [Advanced Compile Options](#advanced-compile-options).

#### CPU mining performance 

Performance is nearly identical to the closed source paid miners. Here are some numbers:

* **I7-2600K** - 266 H/s
* **I7-6700** - 276 H/s (with a separate GPU miner)
* **Dual X5650** - 466 H/s (depends on NUMA)
* **Dual E5640** - 365 H/s (same as above)
* **2X INTEL XEON CPU E5-2699 V4 @ 2.20GHZ** - 1910 H/s

## Default dev donation
By default the miner will donate 1% of the hashpower (1 minute in 100 minutes) to my pool. If you want to change that, edit **donate-level.h** before you build the binaries.

If you want to donate directly to support further development, here is my wallet

fireice-uk:
```
4581HhZkQHgZrZjKeCfCJxZff9E3xCgHGF25zABZz7oR71TnbbgiS7sK9jveE6Dx6uMs2LwszDuvQJgRZQotdpHt1fTdDhk
```

psychocrypt:
```
43NoJVEXo21hGZ6tDG6Z3g4qimiGdJPE6GRxAmiWwm26gwr62Lqo7zRiCJFSBmbkwTGNuuES9ES5TgaVHceuYc4Y75txCTU
```

## Common Issues

**SeLockMemoryPrivilege failed**

Please see [config.txt](config.txt) under section **LARGE PAGE SUPPORT**

For Windows 7 pro, or Windows 8 and above see [this article](https://msdn.microsoft.com/en-gb/library/ms190730.aspx)  (make sure to reboot afterwards!).

For Windows 7 Home :

1) Download and install [Windows Server 2003 Resource Kit Tools](https://www.microsoft.com/en-us/download/details.aspx?id=17657).  Ignore incompatiablity warning during installation.

2) In cmd or power shell: `ntrights -u %USERNAME% +r SeLockMemoryPrivilege`  (where %USERNAME% is the user that will be running the program.  This command needs to be run as admin)

3) Reboot.

Reference: http://rybkaforum.net/cgi-bin/rybkaforum/topic_show.pl?pid=259791#pid259791

*Warning: do not download ntrights.exe from any other site other then the offical Microsoft download page.*

**VirtualAlloc failed**

If you set up the user rights properly (see above), and your system has 4-8GB of RAM (50%+ use), there is a significant chance that there simply won't be a large enough chunk of contiguous memory because Windows is fairly bad at mitigating memory fragmentation.

If that happens, disable all auto-staring applications and run the miner after a reboot.

**msvcp140.dll and vcruntime140.dll not available errors**

Download and install this [runtime package](https://www.microsoft.com/en-us/download/details.aspx?id=48145) from Microsoft.  *Warning: Do NOT use "missing dll" sites - dll's are exe files with another name, and it is a fairly safe bet that any dll on a shady site like that will be trojaned.  Please download offical runtimes from Microsoft above.*


**Error: MEMORY ALLOC FAILED: mmap failed**

From [config.txt](config.txt):

On Linux you will need to configure large page support `sudo sysctl -w vm.nr_hugepages=128` and increase your
ulimit -l. To do do this you need to add following lines to /etc/security/limits.conf:

    * soft memlock 262144
    * hard memlock 262144
    
Save file.  You WILL need to log out and log back in for these settings to take affect on your user (no need to reboot, just relogin in your session).
   
You can also do it Windows-style and simply run-as-root, but this is NOT recommended for security reasons.

**Illegal instruction (core dumped)**

This typically means you are trying to run it on a CPU that does not have [AES](https://en.wikipedia.org/wiki/AES_instruction_set).  This only happens on older version of miner, new version gives better error message (but still wont' work since your CPU doesn't support the required instructions).


## Advanced Compile Options

The build system is CMake, if you are not familiar with CMake you can learn more [here](https://cmake.org/runningcmake/).

### Short Description

There are two easy ways to set variables for `cmake` to configure *xmr-stak-cpu*
- use the ncurses GUI
  - `ccmake .`
  - edit your options
  - end the GUI by pressing the key `c`(create) and than `g`(generate)
- set Options on the command line
  - enable a option: `cmake . -DNAME_OF_THE_OPTION=ON`
  - disable a option `cmake . -DNAME_OF_THE_OPTION=OFF`
  - set a value `cmake . -DNAME_OF_THE_OPTION=value`

After the configuration you need to call
`make install` for slow sequential build
or
`make -j install` for faster parallel build
and install.

### xmr-stak-cpu Compile Options
- `CMAKE_INSTALL_PREFIX` install miner to the home folder
  - `cmake . -DCMAKE_INSTALL_PREFIX=$HOME/xmr-stak-cpu`
  - you can find the binary and the `config.txt` file after `make install` in `$HOME/xmr-stak-cpu/bin`
- `CMAKE_LINK_STATIC` link libgcc and libstdc++ libraries static (default OFF)
  - disable with `cmake . -DCMAKE_LINK_STATIC=ON`
-`CMAKE_BUILD_TYPE` set the build type
  - valid options: `Release` or `Debug`
  - you should always keep `Release` for your productive miners
- `MICROHTTPD_REQUIRED` allow to disable/enable the dependency *microhttpd*
  - by default enabled
  - there is no *http* interface available if option is disabled: `cmake . -DMICROHTTPD_REQUIRED=OFF`
- `OpenSSL_REQUIRED`allow to disable/enable the dependency *OpenSSL*
  - by default enabled
  - it is not possible to connect to a *https* secured pool if optin is disabled: `cmake . -DOpenSSL_REQUIRED=OFF`

