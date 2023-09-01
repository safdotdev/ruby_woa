# ruby_woa


```ps1
scoop bucket add ruby_woa https://github.com/safdotdev/ruby_woa
scoop install ruby_woa
```


## Msys2 ClangARM64

1. `scoop install msys2`
2. Setup Windows terminal for msys2 by adding the following

```json
{
                "commandline": "C:\\Users\\<<your_username>>\\scoop\\apps\\msys2\\current\\msys2_shell.cmd -defterm -no-start -clangarm64 -shell bash",
                "guid": "{0caa0dad-35be-5f56-a8ff-afceeeaa6102}",
                "hidden": false,
                "name": "msys2_clangarm64"
            },
```

3. `pacman -Suy`
4. `pacman -S git`
5. `pacman -S mingw-w64-clang-aarch64-clang`
6. We need a version of ruby to install ruby `pacman -Sy mingw-w64-clang-x86_64-ruby`
  1. Will be in path `/clang64/bin/ruby` 

## Ruby - From Source

Get the source and prep

```
git clone https://github.com/ruby/ruby.git
cd ruby
./autogen.sh
mkdir build && cd build
mkdir ~/.rubies
```

Configure and install

```sh
../configure --prefix "$HOME/.rubies/ruby-master" --with-baseruby=/clang64/bin/ruby
make miniruby
make install
```

## Ruby - From MINGW-packages

```
git clone https://github.com/msys2/MINGW-packages
cd MINGW-packages
cd mingw-w64-ruby
MINGW_ARCH=clangarm64 makepkg-mingw -sLf
```

## Ruby - From MSYS2-packages

```
git clone https://github.com/msys2/MSYS2-packages.git
cd MSYS2-packages
cd ruby
makepkg
```

## Ruby - From rubyinstaller2-packages

```
git clone https://github.com/oneclick/rubyinstaller2-packages.git
cd rubyinstaller2-packages
cd mingw-w64-ruby32
MINGW_ARCH=clangarm64 makepkg-mingw -sLf
```


### Issues

#### 1. Unexpected urctbase.dll when emulating x86/x86_64

Windows On Arm utilising either x86 or x86_64 ruby wil suffer issues.

See [issue](https://github.com/oneclick/rubyinstaller2/issues/308) raised in rubyinstaller2

- x86_64
  - User will experience $EXITCODE of -1073741515 without following step.
  - Requires installing of the [Microsoft Visual C++ Redistributable for Visual Studio 2019 for ARM64.](https://aka.ms/vs/16/release/VC_redist.arm64.exe)
- x86,
  - see [post](https://patriksvensson.se/posts/2020/05/targeting-arm-for-windows-in-rust) - note is for rust, but same for Ruby apps.
  - User will experience error unexpected ucrtbase.dll without following step.
  - Requires installing of insiders build [25250](https://blogs.windows.com/windows-insider/2022/11/28/announcing-windows-11-insider-preview-build-25252/) or later, see [issue](https://github.com/msys2/MINGW-packages/issues/10896)

#### 2. Unexpected urctbase.dll when running native aarch64 ruby builds

- clangarm64 attempt here
 - https://github.com/msys2/MINGW-packages/pull/13115

Same issues with urctbase.dll, need to follow instructions to calculate `__pioinfo`

```console
objdump --disassemble-symbols=_isatty /c/windows/system32/ucrtbase.dll

0x1801e1000 + 0xda0 - 0x180000000 =
0x1E1DA0

cc -DIOINFO_RVA=0x1E1DA0 -o PokeIoInfo.exe PokeIoInfo.c


$ ./PokeIoInfo.exe
matched (0): 540 540
matched (1): 548 548
matched (2): 548 548
matched (3): -1 -1
matched (4): -1 -1
```
