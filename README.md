# exe2dll

A utility for converting Windows PE executables (`.exe`) into dynamic-link libraries (`.dll`) by patching PE headers and injecting export directories.

## Overview

exe2dll enables EXE files to be loaded and invoked as DLLs by:
- Patching PE headers to set DLL flags
- Injecting a DllMain stub and export directory
- Preserving the original entry point as an exported function
- Utilizing existing code caves (no section additions)

## Prerequisites

- Windows PE executable with `.reloc` section
- [peforge](https://github.com/scrymastic/peforge) library for PE manipulation

## Installation

```bash
git clone --recurse-submodules https://github.com/scrymastic/exe2dll.git
build.bat
```

## Usage

```bash
exe2dll.exe <input_exe> <output_dll> <exported_function_name>
```

Example:
```bash
exe2dll.exe test.exe test.dll Run
```
This creates `test.dll` with an exported `Run()` function that points to `test.exe`'s original entry point.

## Technical Details

### Conversion Process
```mermaid
flowchart TD
    A[Start: Input .exe] --> B{Valid PE?}
    B -- No --> X1[Abort: Invalid EXE]
    B -- Yes --> C{Has relocations?}
    C -- No --> X2[Abort: Not safe to convert]
    C -- Yes --> D[Set IMAGE_FILE_DLL flag]
    D --> E[Insert DllMain stub into code cave]
    E --> F[Patch Entry Point to DllMain]
    F --> G[Insert export name string]
    G --> H[Insert DLL name string]
    H --> I[Insert export address table entries]
    I --> J[Insert IMAGE_EXPORT_DIRECTORY]
    J --> K[Patch export directory RVA in PE header]
    K --> L[Write to <output>.dll]
    L --> M[Done: EXE converted to DLL]
```

### Example Output

Original EXE:
![Original EXE structure](imgs/image-1.png)

Converted DLL:
![Converted DLL structure](imgs/image-2.png)

DLL Loading:
![DLL loading demonstration](imgs/image-3.png)

Export Table:
![Export table view](imgs/image-4.png)

## License

See [LICENSE](LICENSE) file for details.
