PEDetour
========
modify binary Portable Executable to hook its export functions  

## Dependencies

This project uses *Capstone* disassembly framework and *Keystone* assembly framework.  
Their licenses and compiled binaries are included in the *capstone-win32* and *keystone-win32* folders.  
Further information are available at [Capstone](http://www.capstone-engine.org) and [Keystone](http://www.keystone-engine.org).  
[LLVM](http://llvm.org/)'s license is also included as a part of *Capstone* and *Keystone*.  

## Compile

This project uses relative library paths so you don't have to adjust paths.  
SDK version is set to 10.0.14393.0 with Toolset Visual Studio 2017 (v141).  
You do not need any modifications if you are using the exact same SDK version and Toolset.  
Otherwise, you might need to recompile capstone and keystone for your specific toolset.  

#### Platform 

There are two platforms available in the VisualStudio solution: *x86* and *x64(x86_64)*  
*x86*: The binary compiled will only support 32-bit PE files, it will throw an exception for 64-bit files.  
*x64(x86_64)*: The binary compiled will only support 64-bit PE files, it will throw an exception for 32-bit files.  

## Usage

PEDetour currently supports two usages: *viewExports* and *injectFunction*  

#### viewExports (Print all export functions listed in the Export Directory)

```
PEDetour PEFileName
PEFileName          the PE file you want to look at
```

```
PEDetour C:\Windows\System32\kernel32.dll   // this is a 64-bit file if you don't know :)
```

#### injectFunction (Inject a piece of code to replace the original export function)

```
PEDetour inputFileName outputFileName functionToInject InjectFileName ...
inputFileName       the PE file you want to inject to (this file itself will not be modified)
outputFileName      where the modified PE will be written to
functionToInject    the function you want to inject (its name as it appears in *viewExports*)
InjectFileName      the assembly file to replace the original function in intel assembly format  
...                 OPTIONAL additional imports you want to add into the output PE  
```

Additional imports must be in the format of "filename.whatever::functionName" (import file name must be in lower cases, functionName is case sensitive).  
If this field is not specified, the following functions will be imported by default,  
* kernel32.dll::GetProcessHeap
* kernel32.dll::HeapAlloc
* kernel32.dll::LoadLibraryA
* kernel32.dll::GetProcAddress
* kernel32.dll::Beep
* user32.dll::MessageBoxA
* inputFileName::functionToInject

```
PEDetour TestDLL.bak TestDLL.dll ?fnTestDLL@@YAHXZ inject.x86.asm   // you can find this in Release binaries
```

## Demo

For notes on injection assembly files, see [inject.x86.asm](https://github.com/chen-charles/PEDetour/blob/master/PEDetour/inject.x86.asm) and [inject.x86_64.asm](https://github.com/chen-charles/PEDetour/blob/master/PEDetour/inject.x86_64.asm)  
You can find demos in [Release](https://github.com/chen-charles/PEDetour/releases) binaries. 

## License

[Version 3 of the GNU General Public Licence (GPLv3)](https://github.com/chen-charles/PEDetour/blob/master/LICENSE)  

## Known Limitations

As of [v1.0](https://github.com/chen-charles/PEDetour/releases/tag/v1.0):  
* *IMAGE_DIRECTORY_ENTRY_SECURITY*, *IMAGE_DIRECTORY_ENTRY_DEBUG*, *IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG* and *IMAGE_DIRECTORY_ENTRY_EXCEPTION* are dropped from the data directories  
* *IMAGE_DIRECTORY_ENTRY_GLOBALPTR*, *IMAGE_DIRECTORY_ENTRY_TLS*, *IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT*, *IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT*, and *IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR* are not fixed  
* there is an issue with capstone(3.0.4) and keystone(0.9.1) for x86_64 REX prefix handling, see line 200 in [PE.h](https://github.com/chen-charles/PEDetour/blob/master/PEDetour/PE.h) (code generated by Visual Studio), and is currently bypassed through hard coding (ikr)  
* for x86_64, you must use full 64-bit addressing for library function calls (as described in [inject.x86_64.asm](https://github.com/chen-charles/PEDetour/blob/master/PEDetour/inject.x86_64.asm))  
* .xdata sections are not available for injected function, you must use either the stack space, or *kernel32::GetProcessHeap* with *kernel32::HeapAlloc*  
