# Background
This project reproduces an error where the abseil library is built with the MSVC compiler and the program that links to the abseil library is built with clang/LLVM.
When abseil is build with MSVC it generates CMake targets with the property INTERFACE_LINK_LIBRARIES set with flags not compatible with clang/LLVM, **-ignore:4221**.
This makes it impossible to link to the MSVC built abseil library with clang/LLVM.

For example see the generated file abslTargets.cmake:
```
set_target_properties(absl::atomic_hook PROPERTIES
  INTERFACE_INCLUDE_DIRECTORIES "${_IMPORT_PREFIX}/include"
  INTERFACE_LINK_LIBRARIES "absl::config;absl::core_headers;-ignore:4221"
)
```

# Prerequisites
* Windows build environment with the following installed:
  - Visual Sudio and LLVM installed. My current LLVM version is 17.0.6 and MSVC version is 19.38.33134 but this is most probably reproducable with the latest versions of these tools.
  - CMake
  - Ninja

# How to Reproduce
This example is using abseil from VCPKG where VCPKG will build the libraries with MSVC and the example is explicitly built with clang/LLVM.
Make sure to clear your VCPKG cache first.

Steps to reproduce:
```
git clone https://github.com/kotten73/abseil_repro
cd abseil_repro
mkdir build
cd build
cmake -G "Ninja" .. -DCMAKE_C_COMPILER="clang" -DCMAKE_CXX_COMPILER="clang++" -DCMAKE_RC_COMPILER="llvm-rc"
ninja
```
The error I get is:

```
D:\Projects\abseil_repro\build [main ≡ +1 ~0 -0 !]> ninja
[2/2] Linking CXX executable abseil_repro.exe
FAILED: abseil_repro.exe
C:\WINDOWS\system32\cmd.exe /C "cd . && C:\Users\maka\scoop\apps\llvm\current\bin\clang++.exe -nostartfiles -nostdlib -O0 -D_DEBUG -D_DLL -D_MT -Xclang --dependent-lib=msvcrtd -g -Xclang -gcodeview -Xlinker /subsystem:console  -fuse-ld=lld-link CMakeFiles/abseil_repro.dir/src/main.cpp.obj -o abseil_repro.exe -Xlinker /MANIFEST:EMBED -Xlinker /implib:abseil_repro.lib -Xlinker /pdb:abseil_repro.pdb -Xlinker /version:0.0   vcpkg_installed/x64-windows-static/debug/lib/absl_bad_any_cast_impl.lib  vcpkg_installed/x64-windows-static/debug/lib/absl_raw_logging_internal.lib  vcpkg_installed/x64-windows-static/debug/lib/absl_log_severity.lib  -ignore:4221  -lkernel32 -luser32 -lgdi32 -lwinspool -lshell32 -lole32 -loleaut32 -luuid -lcomdlg32 -ladvapi32 -loldnames  && C:\WINDOWS\system32\cmd.exe /C "cd /D D:\Projects\abseil_repro\build && "C:\Program Files\PowerShell\7\pwsh.exe" -noprofile -executionpolicy Bypass -file D:/Projects/vcpkg/scripts/buildsystems/msbuild/applocal.ps1 -targetBinary D:/Projects/abseil_repro/build/abseil_repro.exe -installedDir D:/Projects/abseil_repro/build/vcpkg_installed/x64-windows-static/debug/bin -OutVariable out""
clang++: error: unknown argument: '-ignore:4221'
ninja: build stopped: subcommand failed.
```


