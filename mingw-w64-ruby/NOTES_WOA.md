## Ruby Aarch64 Windows Builds - wip notes

Aarch64-w64-mingw32

* Download/Install MSYS2 as usual
* Execute clangarm64.exe in the MSYS2 install directory
* Run pacman -Suy
* Install clang for example: pacman -S mingw-w64-clang-aarch64-clang

https://www.msys2.org/wiki/arm64/
https://github.com/msys2/MINGW-packages/blob/master/mingw-w64-ruby/PKGBUILD

https://github.com/oneclick/rubyinstaller2-packages

Builds 

mingw-w64-ucrt-x86_64-${package-name}

makedepends=("${MINGW_PACKAGE_PREFIX}-cc"
             "${MINGW_PACKAGE_PREFIX}-autotools"
             "${MINGW_PACKAGE_PREFIX}-doxygen"
             "${MINGW_PACKAGE_PREFIX}-graphviz")
depends=("${MINGW_PACKAGE_PREFIX}-gcc-libs"
         "${MINGW_PACKAGE_PREFIX}-gdbm"
         "${MINGW_PACKAGE_PREFIX}-libyaml"
         "${MINGW_PACKAGE_PREFIX}-libffi"
         "${MINGW_PACKAGE_PREFIX}-pdcurses"
         "${MINGW_PACKAGE_PREFIX}-openssl"
         "${MINGW_PACKAGE_PREFIX}-gmp"
         "${MINGW_PACKAGE_PREFIX}-tk")


pacman -S mingw-w64-clang-aarch64-autotools

https://packages.msys2.org/package/mingw-w64-clang-aarch64-libffi?repo=clangarm64

pacman -Sy git
pacman -Sy mingw-w64-clang-aarch64-clang
pacman -Sy mingw-w64-clang-aarch64-cc
pacman -Sy mingw-w64-clang-aarch64-autotools
pacman -Sy mingw-w64-clang-aarch64-doxygen
pacman -Sy mingw-w64-clang-aarch64-graphviz
pacman -S mingw-w64-clang-aarch64-gcc-libs
pacman -Sy mingw-w64-clang-aarch64-gdbm
pacman -Sy mingw-w64-clang-aarch64-libyaml
pacman -Sy mingw-w64-clang-aarch64-libffi
pacman -Sy mingw-w64-clang-aarch64-pdcurses
pacman -Sy mingw-w64-clang-aarch64-openssl
pacman -Sy mingw-w64-clang-aarch64-gmp
pacman -Sy mingw-w64-clang-aarch64-tk


Crystal
pacman -Sy mingw-w64-llvm-15

   10  pacman -S mingw-w64-clang-aarch64-clang
   14  pacman -Sy git
   18  clang --version
   21  git clone https://github.com/ruby/ruby
   22  cd ruby
   26  ./autogen.sh
   27  mkdir build
   28  cd build
   29  mkdir ~/.rubies
   34  pacman -Sy mingw-w64-clang-x86_64-ruby
   37  /clang64/bin/ruby --version
   43  ../configure --prefix "$HOME/.rubies/ruby-master" --with-baseruby=/clang64/bin/ruby
   45  export CFLAGS+=" -Wno-unknown-attributes"
   46  export CXXFLAGS+=" -Wno-unknown-attributes"
   47  make install


export FFI_INC=/clangarm64/include
export MINGW_PREFIX=/clangarm64
export CPPFLAGS+=" -DFD_SETSIZE=2048 ${FFI_INC} -I${MINGW_PREFIX}/include/pdcurses"
export CFLAGS+=" -I${MINGW_PREFIX}/include/pdcurses ${FFI_INC}"
export CXXFLAGS+=" -I${MINGW_PREFIX}/include/pdcurses ${FFI_INC}"


## Unexpected urctbase.dll

https://github.com/msys2/MINGW-packages/pull/13115

https://github.com/msys2/MINGW-packages/pull/13115#issuecomment-1264304357

objdump --disassemble-symbols=_isatty /c/windows/system32/ucrtbase.dll

Work lappy

0x1801d8000 + 0xdb0 - 0x180000000 =
0x1D8DB0


Personal lappy
0x1801e1000 + 0xda0 - 0x180000000 =
0x1E1DA0

cc -DIOINFO_RVA=0x1E1DA0 -o PokeIoInfo.exe PokeIoInfo.c


$ ./PokeIoInfo.exe
matched (0): 540 540
matched (1): 548 548
matched (2): 548 548
matched (3): -1 -1
matched (4): -1 -1


$ src/build-CLANGARM64/miniruby.exe --version
ruby 3.2.2 (2023-03-30 revision e51014f9c0) [aarch64-mingw-ucrt]


More things to add

pacman -S mingw-w64-clang-aarch64-toolchain

../configure --prefix "$HOME/.rubies/ruby-master" --build=${MINGW_CHOST}   --host=${MINGW_CHOST}   --target=${MINGW_CHOST}  --disable-install-doc --disable-werror -with-setjmp-type=setjmp --with-baseruby=/clang64/bin/ruby

We need a base ruby to be able to build


Instructions for finding address

1. I can not find any "marker" bytes like PIOINFO_MARK in _isatty function in AArch64 ucrtbase.dll. So, I went with objdump.
2. Here is the important part in _isatty function from ucrtbase.dll in AArch64 WinPE.
18001072c: f0000e48     adrp    x8, #0x1801db000
180010730: 91370108     add     x8, x8, #0xdc0

* The offset of __pioinfo would be 0x1801db000 + 0xdc0 - 0x180000000 = 0x1dbdc0
* The actual memory address of __pioinfo will be GetModuleHandle("ucrtbase.dll") + 0x1dbdc0
3. I have applied that idea with this simple code https://github.com/Biswa96/Junkyard/blob/master/c/PokeIoInfo.c. And it works with x86_64 and AArch64 both.



pacman -S mingw-w64-clang-aarch64-cc mingw-w64-clang-aarch64-autotools mingw-w64-clang-aarch64-doxygen mingw-w64-clang-aarch64-graphviz mingw-w64-clang-aarch64-gcc-libs mingw-w64-clang-aarch64-gdbm mingw-w64-clang-aarch64-libyaml mingw-w64-clang-aarch64-libffi mingw-w64-clang-aarch64-pdcurses mingw-w64-clang-aarch64-openssl mingw-w64-clang-aarch64-gmp mingw-w64-clang-aarch64-tk


Insiders Canary 25941.1000
0x1801f7000 + 0xe80 - 0x180000000 = 0x1F7E80
cc -DIOINFO_RVA=0x1F7E80 -o PokeIoInfo.exe PokeIoInfo.c x86_64 ruby now works, was seg faulting with unexpected ucrtbase.dll before


## General Ruby Windows on ARM notes

Running on an ARM64 machine
Everything compiled! Not a big surprise, it's "Guaranteed to build" after all, but it felt good anyway. Time to test it out on an actual ARM64 machine.
> ./helloworld.exe
-1073741515
Oh noes
The application exited with the error code -1073741515, more famously known as 0xC0000135, which is Windows' way of telling us that an essential component is missing. It turns out that we need to install the Microsoft Visual C++ Redistributable for Visual Studio 2019 for ARM64.
Installing the MSVC redistributable and rerunning our app now produces the expected output.
> ./helloworld.exe
Hello World


PS C:\Users\saf> Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsHardwareAbstractionLayerVersion

WindowsProductName WindowsVersion OsHardwareAbstractionLayerVersion
------------------ -------------- ---------------------------------
Windows 10 Pro     2009


PS C:\Users\saf> [System.Environment]::OSVersion.Version
>>

Major  Minor  Build  Revision
-----  -----  -----  --------
10     0      22621  0



Downloading

2023-03 Cumulative Update Preview for Windows 11 Version 22H2 for arm64-based Systems (KB5023778)
https://blogs.windows.com/windows-insider/2023/04/07/announcing-windows-11-insider-preview-build-23430/



Ruby errors

https://bugs.ruby-lang.org/issues/18605

32bit ruby using ucrt has started to fail on newer Windows with "unexpected ucrtbase.dll" -> https://github.com/ruby/ruby/blob/3fb7d2cadc18472ec107b14234933b017a33c14d/win32/win32.c#L2591
The problem is that ruby depends on ucrt internals and those have apparently changed with newer versions.
See https://github.com/msys2/MINGW-packages/pull/10878 and https://github.com/msys2/MINGW-packages/issues/10896 for some background and a potential fix. But ideally ruby wouldn't depend on Windows internals like this.


Need to try win 10 thanks  https://github.com/oneclick/rubyinstaller2/issues/308

@ArminG-MSFT 25250 is now working for me on Windows on Arm Dev Kit 2023 device - thanks!


https://blogs.windows.com/windows-insider/2022/11/28/announcing-windows-11-insider-preview-build-25252/


Node js compatible ffi layer for arm64 windows

koffi - npm (npmjs.com)


