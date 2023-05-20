# Implementing A Function from BLAS library using Halide and C

In this project , I provide two implementations of the function SGEMV() from BLAS library, one using Halide and the other using C.

## Table of Contents

- [Building Halide](#Building_Halide)
- [Usage](#usage)
- [Documentation](#documentation)
- [Configuration](#configuration)
- [Contributing](#contributing)
- [Testing](#testing)
- [License](#license)
- [Changelog](#changelog)
- [Credits](#credits)
- [Contact](#contact)

## Building Halide

To work on this project, I built Halide from sources using make. To do that I executed this commands:
```bash
$ git clone --depth 1 --branch llvmorg-14.0.0 https://github.com/llvm/llvm-project.git
$ cmake -DCMAKE_BUILD_TYPE=Release \
      -DLLVM_ENABLE_PROJECTS="clang;lld;clang-tools-extra" \
      -DLLVM_TARGETS_TO_BUILD="X86;ARM;NVPTX;AArch64;Hexagon;WebAssembly" \
      -DLLVM_ENABLE_TERMINFO=OFF -DLLVM_ENABLE_ASSERTIONS=ON \
      -DLLVM_ENABLE_EH=ON -DLLVM_ENABLE_RTTI=ON -DLLVM_BUILD_32_BITS=OFF \
      -S llvm-project/llvm -B llvm-build
$ cmake --build llvm-build
$ cmake --install llvm-build --prefix llvm-install

$ export LLVM_ROOT=$PWD/llvm-install
$ export LLVM_CONFIG=$LLVM_ROOT/bin/llvm-config

$ git clone https://github.com/halide/Halide.git
$ mkdir halide_build
$ cd halide_build
$ make -f ../Halide/Makefile        #Or moving to the halide directory and execut $ make
$ make run_tests    #just for tests
$ make test_apps    #just for test
```
In my case I first built halide in a new directory (halide-build) than I prefered building in the same directory so I used the second method. All the the
text printed in the screen while executing this script are in the text file called logs , to access directy to the begining of the commands you can use : CTRL+F and type  
```bash 
----------------------------------
```
### Some Problems
When executing the command make to build halide I got this error : 
```bash
Documents/Halide/src/CodeGen_Internal.cpp: In function ‘void Halide::Internal::embed_bitcode(llvm::Module*, const string&)’:
Documents/Halide/src/CodeGen_Internal.cpp:739:24: error: ‘static void llvm::GlobalVariable::operator delete(void*)’ called on pointer returned from a mismatched allocation function [-Werror=mismatched-new-delete]
739 | module_constant);
| ^
In file included from Documents/llvm-install/include/llvm/IR/Module.h:29,
from Documents/llvm-install/include/llvm/IR/PassManager.h:46,
from Documents/llvm-install/include/llvm/Analysis/AliasAnalysis.h:45,
from Documents/Halide/src/LLVM_Headers.h:35,
from Documents/Halide/src/CodeGen_Internal.cpp:7:
Documents/llvm-install/include/llvm/IR/GlobalVariable.h:73:30: note: returned from ‘static void* llvm::User::operator new(size_t, unsigned int)’
73 | return User::operator new(s, 1);
| ~~~~~~~~~~~~~~~~~~^~~~~~
Documents/Halide/src/CodeGen_Internal.cpp:759:56: error: ‘static void llvm::GlobalVariable::operator delete(void*)’ called on pointer returned from a mismatched allocation function [-Werror=mismatched-new-delete]
759 | command_line_constant);
| ^
In file included from Documents/llvm-install/include/llvm/IR/Module.h:29,
from Documents/llvm-install/include/llvm/IR/PassManager.h:46,
from Documents/llvm-install/include/llvm/Analysis/AliasAnalysis.h:45,
from Documents/Halide/src/LLVM_Headers.h:35,
from Documents/Halide/src/CodeGen_Internal.cpp:7:
Documents/llvm-install/include/llvm/IR/GlobalVariable.h:73:30: note: returned from ‘static void* llvm::User::operator new(size_t, unsigned int)’
73 | return User::operator new(s, 1);
| ~~~~~~~~~~~~~~~~~~^~~~~~
Documents/Halide/src/CodeGen_Internal.cpp:778:76: error: ‘static void llvm::GlobalVariable::operator delete(void*)’ called on pointer returned from a mismatched allocation function [-Werror=mismatched-new-delete]
778 | llvm::ConstantArray::get(ATy, used_array), "llvm.compiler.used");
| ^
In file included from Documents/llvm-install/include/llvm/IR/Module.h:29,
from Documents/llvm-install/include/llvm/IR/PassManager.h:46,
from Documents/llvm-install/include/llvm/Analysis/AliasAnalysis.h:45,
from Documents/Halide/src/LLVM_Headers.h:35,
from Documents/Halide/src/CodeGen_Internal.cpp:7:
Documents/llvm-install/include/llvm/IR/GlobalVariable.h:73:30: note: returned from ‘static void* llvm::User::operator new(size_t, unsigned int)’
73 | return User::operator new(s, 1);
| ~~~~~~~~~~~~~~~~~~^~~~~~
At global scope:
cc1plus: note: unrecognized command-line option ‘-Wno-unknown-warning-option’ may have been intended to silence earlier diagnostics
cc1plus: all warnings being treated as errors
make: *** [../Halide/Makefile:1213: bin/build/CodeGen_Internal.o] Error 1
```
I solved it by removing the -Werror flag from the makfile of halide.
Another Error that I got is when compiling a halide program , the error looks like this:

```bash 
/usr/bin/ld: /home/kasmi/Documents/Halide/lib/libHalide.a(llvm_1351_Process.cpp.o): in function llvm::sys::Process::FileDescriptorHasColors(int)': (.text._ZN4llvm3sys7Process23FileDescriptorHasColorsEi+0x69): undefined reference to set_curterm'
/usr/bin/ld: (.text._ZN4llvm3sys7Process23FileDescriptorHasColorsEi+0x82): undefined reference to setupterm' /usr/bin/ld: (.text._ZN4llvm3sys7Process23FileDescriptorHasColorsEi+0x92): undefined reference to tigetnum'
/usr/bin/ld: (.text._ZN4llvm3sys7Process23FileDescriptorHasColorsEi+0x9f): undefined reference to set_curterm' /usr/bin/ld: (.text._ZN4llvm3sys7Process23FileDescriptorHasColorsEi+0xa7): undefined reference to del_curterm'
/usr/bin/ld: /home/kasmi/Documents/Halide/lib/libHalide.a(llvm_1306_Compression.cpp.o): in function llvm::zlib::compress(llvm::StringRef, llvm::SmallVectorImpl<char>&, int)': (.text._ZN4llvm4zlib8compressENS_9StringRefERNS_15SmallVectorImplIcEEi+0x2f): undefined reference to compressBound'
/usr/bin/ld: (.text._ZN4llvm4zlib8compressENS_9StringRefERNS_15SmallVectorImplIcEEi+0x70): undefined reference to compress2' /usr/bin/ld: /home/kasmi/Documents/Halide/lib/libHalide.a(llvm_1306_Compression.cpp.o): in function llvm::zlib::uncompress(llvm::StringRef, char*, unsigned long&)':
(.text._ZN4llvm4zlib10uncompressENS_9StringRefEPcRm+0x2f): undefined reference to uncompress' /usr/bin/ld: /home/kasmi/Documents/Halide/lib/libHalide.a(llvm_1306_Compression.cpp.o): in function llvm::zlib::uncompress(llvm::StringRef, llvm::SmallVectorImpl&, unsigned long)':
(.text._ZN4llvm4zlib10uncompressENS_9StringRefERNS_15SmallVectorImplIcEEm+0x66): undefined reference to uncompress' /usr/bin/ld: /home/kasmi/Documents/Halide/lib/libHalide.a(llvm_1306_Compression.cpp.o): in function llvm::zlib::crc32(llvm::StringRef)':
(.text._ZN4llvm4zlib5crc32ENS_9StringRefE+0x9): undefined reference to crc32' /usr/bin/ld: /home/kasmi/Documents/Halide/lib/libHalide.a(llvm_1310_CRC.cpp.o): in function llvm::crc32(unsigned int, llvm::ArrayRef)':
(.text._ZN4llvm5crc32EjNS_8ArrayRefIhEE+0x34): undefined reference to crc32' /usr/bin/ld: /home/kasmi/Documents/Halide/lib/libHalide.a(llvm_1310_CRC.cpp.o): in function llvm::crc32(llvm::ArrayRef)':
(.text._ZN4llvm5crc32ENS_8ArrayRefIhEE+0x34): undefined reference to crc32' /usr/bin/ld: /home/kasmi/Documents/Halide/lib/libHalide.a(llvm_1310_CRC.cpp.o): in function llvm::JamCRC::update(llvm::ArrayRef)':
(.text._ZN4llvm6JamCRC6updateENS_8ArrayRefIhEE+0x34): undefined reference to `crc32'
collect2: error: ld returned 1 exit status
```
I solved this problem by adding this parameter to the command of compilation: 
```bash
-lz -ltinfo
```

## Usage

Detailed instructions on how to use the project. Include examples or code snippets to demonstrate common use cases. Explain any necessary configuration or environment variables.

## Documentation

If your project has extensive documentation, provide a brief description here and provide links to the relevant resources, such as user guides, API references, or wiki pages.

## Configuration

Explain how to configure the project, including information about configuration files, environment variables, or any other relevant settings.

## Contributing

If you want to encourage contributions from the community, outline guidelines for how others can contribute to the project. Specify any rules or processes for submitting pull requests or reporting issues.

## Testing

Explain how to run the tests and provide any necessary details about the testing framework or environment.

## License

Specify the project's license and provide a copy of the license file if applicable. Make sure to comply with any licensing requirements.

## Changelog

Highlight notable changes in each version of the project. You can use headings, bullet points, or tables to present the changelog.

## Credits

Acknowledge any external libraries, tools, or resources that were used in the project. Give proper credit to the original authors or contributors.

## Contact

Provide contact information for the project maintainer or team, such as email addresses or links to relevant communication channels.
