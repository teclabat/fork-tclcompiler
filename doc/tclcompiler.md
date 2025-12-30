# Tcl Bytecode Compiler (tclcompiler)

A Tcl extension for compiling Tcl scripts into bytecode files for distribution and faster loading.

**Version:** 2.0a0 \
**Package:** `tclcompiler` \
**Namespace:** `compiler::`

---

## Table of Contents

1. [Overview](#overview)
2. [Features](#features)
3. [Requirements](#requirements)
4. [Installation](#installation)
5. [Quick Start](#quick-start)
6. [Command Reference](#command-reference)
   - [Compilation Commands](#compilation-commands)
   - [Utility Commands](#utility-commands)
7. [Usage Examples](#usage-examples)
8. [Compilation Options](#compilation-options)
9. [Output Files](#output-files)
10. [Error Handling](#error-handling)
11. [Performance Considerations](#performance-considerations)
12. [Security Considerations](#security-considerations)
13. [Best Practices](#best-practices)
14. [Troubleshooting](#troubleshooting)
15. [Building from Source](#building-from-source)
16. [Platform Support](#platform-support)
17. [License](#license)
18. [Version History](#version-history)
19. [Additional Resources](#additional-resources)

---

## Overview

The **tclcompiler** package provides commands to compile Tcl scripts into Tcl's internal bytecode format. The resulting `.tbc` (Tcl ByteCode) files can be distributed to end users and loaded using the companion **tbcload** package.

Bytecode compilation enables you to:

- **Protect source code** - Distribute applications without exposing readable Tcl source
- **Improve startup performance** - Bypass the parsing phase during script loading
- **Package applications** - Create deployable applications with obscured source
- **Reduce file count** - Combine multiple scripts into compiled modules

The compiler analyzes Tcl scripts, compiles procedures into bytecode, and writes the result to `.tbc` files that can be loaded transparently by applications using `tbcload`.

## Features

- **Full Tcl 8.6 and 9.0 support** - Compatible with modern Tcl versions
- **Automatic output naming** - Generates `.tbc` files automatically if no output specified
- **Preamble injection** - Add initialization code to compiled files
- **Procedure compilation** - All procedures are compiled to bytecode
- **Namespace support** - Correctly handles namespaced procedures and commands
- **Error preservation** - Maintains line number information for debugging
- **Cross-platform** - Works on Windows, Linux, macOS, and Unix systems
- **TEA-compliant** - Standard Tcl Extension Architecture build system

## Requirements

### Software Requirements

- **Tcl 8.6 or later** or **Tcl 9.0** - Required for compilation
- **C compiler** - For building from source (GCC, Clang, MSVC)

### Runtime Requirements

Compiled bytecode requires:

- **tbcload package** - To load and execute `.tbc` files
- **Matching Tcl version** - Bytecode compiled for Tcl 8.x requires Tcl 8.x runtime (same for 9.x)

### Important Version Compatibility

**Critical:** Bytecode is version-specific:

- Code compiled with tclcompiler for **Tcl 8.6** can only run on **Tcl 8.6+** with tbcload
- Code compiled with tclcompiler for **Tcl 9.0** can only run on **Tcl 9.0+** with tbcload
- **Cross-version loading is not supported**

### Supported Platforms

- Windows (32-bit and 64-bit)
- Linux (32-bit and 64-bit)
- macOS
- BSD (FreeBSD, OpenBSD, NetBSD)
- Unix (Solaris, AIX, HP-UX)

---

## Installation

### Using Package Managers

**Tcl Dev Kit:**
Many Tcl Dev Kit distributions include tclcompiler.

**Linux distributions:**
```bash
# Fedora/RHEL/CentOS
sudo dnf install tclcompiler

# Debian/Ubuntu (if available)
sudo apt-get install tclcompiler
```

### From Binary Distribution

1. Obtain pre-built tclcompiler for your platform and Tcl version
2. Extract to a directory on your Tcl `auto_path`
3. Verify installation with `package require tclcompiler`

### Verifying Installation

```tcl
package require tclcompiler
puts "tclcompiler version: [package present tclcompiler]"
puts "Compiled against Tcl: [compiler::getTclVer]"
```

---

## Quick Start

### Compile a Simple Script

**Step 1:** Create a Tcl script

```tcl
# hello.tcl
proc greet {name} {
    puts "Hello, $name!"
}

greet "World"
```

**Step 2:** Compile to bytecode

```tcl
package require tclcompiler
compiler::compile hello.tcl
# Creates hello.tbc automatically
```

**Step 3:** Run the bytecode

```tcl
package require tbcload
source hello.tbc
# Output: Hello, World!
```

---

## Command Reference

### Compilation Commands

#### `compiler::compile`

Compiles a Tcl script file into bytecode format.

**Syntax:**
```tcl
compiler::compile inputFile ?outputFile?
compiler::compile -preamble preambleCode inputFile outputFile
```

**Parameters:**

- `inputFile` - Path to the Tcl source file to compile (.tcl)
- `outputFile` - (Optional) Path for the output bytecode file
  - If omitted, uses input filename with `.tbc` extension
  - Example: `script.tcl` → `script.tbc`
- `-preamble preambleCode` - (Optional) Code to execute before the compiled script
  - Useful for setting variables or initialization
  - Must be followed by both inputFile and outputFile

**Returns:** Nothing on success; raises error on failure

**Behavior:**

1. Reads and parses the input Tcl script
2. Compiles all procedure bodies to bytecode
3. Preserves line number information for error reporting
4. Writes bytecode to output file
5. Maintains namespace structure

**Example:**
```tcl
# Basic compilation (output: script.tbc)
compiler::compile script.tcl

# Explicit output file
compiler::compile script.tcl /output/script.tbc

# With preamble
compiler::compile -preamble {
    set ::appVersion "1.0"
    set ::debug 0
} app.tcl app.tbc
```

**Notes:**

- Input file must exist and be readable
- Output directory must exist and be writable
- Existing output files are overwritten without warning
- Compilation preserves procedure definitions but not top-level code execution

---

### Utility Commands

#### `compiler::getBytecodeExtension`

Returns the file extension used for compiled bytecode files.

**Syntax:**
```tcl
set ext [compiler::getBytecodeExtension]
```

**Parameters:** None

**Returns:** String containing the bytecode file extension (typically `.tbc`)

**Example:**
```tcl
set ext [compiler::getBytecodeExtension]
puts "Bytecode extension: $ext"
# Output: Bytecode extension: .tbc

# Use in file operations
set sourceFile "myapp.tcl"
set bytecodeFil [file rootname $sourceFile]$ext
compiler::compile $sourceFile $bytecodeFile
```

---

#### `compiler::getTclVer`

Returns the Tcl version that the compiler package was built against.

**Syntax:**
```tcl
set version [compiler::getTclVer]
```

**Parameters:** None

**Returns:** Tcl version string (e.g., "8.6", "9.0")

**Use Case:**

This command helps verify version compatibility between the compiler and the target runtime environment.

**Example:**
```tcl
set compilerTclVer [compiler::getTclVer]
set runtimeTclVer [info patchlevel]

puts "Compiler built for: Tcl $compilerTclVer"
puts "Running on: Tcl $runtimeTclVer"

# Check major version compatibility
if {[lindex [split $compilerTclVer .] 0] ne [lindex [split $runtimeTclVer .] 0]} {
    puts "WARNING: Version mismatch - bytecode may not be compatible"
}
```

---

## Usage Examples

### Example 1: Basic Script Compilation

Compile a single script with automatic output naming:

```tcl
package require tclcompiler

# Source file: calculator.tcl
compiler::compile calculator.tcl

# Creates: calculator.tbc in the same directory
puts "Compilation complete: calculator.tbc"
```

---

### Example 2: Batch Compilation

Compile all Tcl scripts in a directory:

```tcl
package require tclcompiler

proc compile_directory {sourceDir {destDir ""}} {
    if {$destDir eq ""} {
        set destDir $sourceDir
    }

    set ext [compiler::getBytecodeExtension]
    set compiled 0
    set failed 0

    foreach tclFile [glob -nocomplain -directory $sourceDir *.tcl] {
        set baseName [file tail [file rootname $tclFile]]
        set tbcFile [file join $destDir "${baseName}${ext}"]

        puts "Compiling: [file tail $tclFile] -> [file tail $tbcFile]"

        if {[catch {
            compiler::compile $tclFile $tbcFile
            incr compiled
        } err]} {
            puts stderr "  ERROR: $err"
            incr failed
        }
    }

    puts "\nCompilation summary:"
    puts "  Success: $compiled files"
    puts "  Failed:  $failed files"
}

# Compile all scripts in src/ to build/
file mkdir build
compile_directory src build
```

---

### Example 3: Compilation with Preamble

Add version information and configuration to compiled output:

```tcl
package require tclcompiler

# Define preamble code
set preamble {
    # Application metadata
    set ::APP_NAME "MyApplication"
    set ::APP_VERSION "1.2.3"
    set ::APP_BUILD_DATE "[clock format [clock seconds]]"

    # Configuration
    set ::DEBUG_MODE 0
    set ::LOG_LEVEL "info"

    # Initialize namespace
    namespace eval ::myapp {
        variable initialized 1
    }
}

# Compile with preamble
compiler::compile -preamble $preamble myapp.tcl myapp.tbc

puts "Application compiled with embedded metadata"
```

**Runtime (in compiled bytecode):**
```tcl
package require tbcload
source myapp.tbc

# Preamble variables are now available
puts "Running: $::APP_NAME version $::APP_VERSION"
puts "Built on: $::APP_BUILD_DATE"
```

---

### Example 4: Multi-Module Application

Compile a structured application with multiple modules:

```tcl
package require tclcompiler

# Directory structure:
# myapp/
#   ├── main.tcl
#   └── lib/
#       ├── database.tcl
#       ├── ui.tcl
#       └── utils.tcl

proc compile_project {srcRoot destRoot} {
    set ext [compiler::getBytecodeExtension]

    # Ensure destination structure exists
    file mkdir $destRoot
    file mkdir [file join $destRoot lib]

    # Compile main application
    compiler::compile \
        [file join $srcRoot main.tcl] \
        [file join $destRoot "main${ext}"]

    # Compile library modules
    foreach module {database ui utils} {
        compiler::compile \
            [file join $srcRoot lib "${module}.tcl"] \
            [file join $destRoot lib "${module}${ext}"]
    }

    puts "Project compiled successfully"
}

# Execute compilation
compile_project myapp myapp-compiled
```

---

### Example 5: Version-Aware Compilation

Check version compatibility before compilation:

```tcl
package require tclcompiler

proc safe_compile {inputFile outputFile} {
    # Get compiler Tcl version
    set compilerVer [compiler::getTclVer]
    set runtimeVer [info patchlevel]

    # Extract major version
    set compilerMajor [lindex [split $compilerVer .] 0]
    set runtimeMajor [lindex [split $runtimeVer .] 0]

    if {$compilerMajor ne $runtimeMajor} {
        error "Version mismatch: Compiler for Tcl $compilerVer, runtime is $runtimeVer. \
               Bytecode will not be compatible."
    }

    # Compile
    puts "Compiling for Tcl $compilerMajor.x: $inputFile"
    compiler::compile $inputFile $outputFile

    puts "Success: $outputFile (compatible with Tcl $compilerMajor.x)"
}

# Use safe compilation
if {[catch {
    safe_compile myapp.tcl myapp.tbc
} err]} {
    puts stderr "Compilation failed: $err"
    exit 1
}
```

---

### Example 6: Build System Integration

Integrate with a Makefile-style build system:

```tcl
# build.tcl - Build script for automated compilation
package require tclcompiler

proc build_target {target} {
    set ext [compiler::getBytecodeExtension]

    switch -exact -- $target {
        "all" {
            build_target "app"
            build_target "tests"
        }
        "app" {
            puts "Building application..."
            file mkdir dist
            foreach src [glob src/*.tcl] {
                set base [file tail [file rootname $src]]
                set dest "dist/${base}${ext}"
                puts "  $src -> $dest"
                compiler::compile $src $dest
            }
        }
        "tests" {
            puts "Building test suite..."
            file mkdir dist/tests
            foreach src [glob tests/*.tcl] {
                set base [file tail [file rootname $src]]
                set dest "dist/tests/${base}${ext}"
                puts "  $src -> $dest"
                compiler::compile $src $dest
            }
        }
        "clean" {
            puts "Cleaning build artifacts..."
            file delete -force dist
        }
        default {
            puts stderr "Unknown target: $target"
            puts "Available targets: all, app, tests, clean"
            exit 1
        }
    }
}

# Run build
if {$argc < 1} {
    set target "all"
} else {
    set target [lindex $argv 0]
}

build_target $target
puts "Build complete: $target"
```

**Usage:**
```bash
tclsh build.tcl all     # Build everything
tclsh build.tcl app     # Build application only
tclsh build.tcl clean   # Remove build artifacts
```

---

### Example 7: Package Creation

Create an installable package with compiled bytecode:

```tcl
# create_package.tcl
package require tclcompiler

proc create_package {pkgName pkgVersion sourceFiles destDir} {
    set ext [compiler::getBytecodeExtension]
    set pkgDir [file join $destDir $pkgName$pkgVersion]

    # Create package directory
    file mkdir $pkgDir

    # Compile all source files
    set compiledFiles {}
    foreach srcFile $sourceFiles {
        set baseName [file tail [file rootname $srcFile]]
        set destFile [file join $pkgDir "${baseName}${ext}"]

        puts "Compiling: $srcFile"
        compiler::compile $srcFile $destFile

        lappend compiledFiles "${baseName}${ext}"
    }

    # Create pkgIndex.tcl
    set indexFile [file join $pkgDir pkgIndex.tcl]
    set fp [open $indexFile w]

    puts $fp "# Package Index for $pkgName $pkgVersion"
    puts $fp "package ifneeded $pkgName $pkgVersion \[list apply {{dir} {"
    puts $fp "    package require tbcload"

    foreach file $compiledFiles {
        puts $fp "    source \[file join \$dir $file\]"
    }

    puts $fp "}} \$dir\]"
    close $fp

    puts "\nPackage created: $pkgDir"
    puts "Files: [llength $compiledFiles] compiled modules"
}

# Create package
create_package MyLib 1.0 {
    src/core.tcl
    src/utils.tcl
    src/api.tcl
} /usr/local/lib/tcl8.6
```

---

## Compilation Options

### Standard Compilation

Basic compilation with automatic output naming:

```tcl
compiler::compile input.tcl
# Creates: input.tbc
```

### Explicit Output Path

Specify custom output location and filename:

```tcl
compiler::compile input.tcl /custom/path/output.tbc
```

### Preamble Code Injection

Add initialization code that runs before the compiled script:

```tcl
compiler::compile -preamble {
    # This code runs first when the .tbc is sourced
    set ::initialized 1
    namespace eval ::myapp {}
} input.tcl output.tbc
```

**Preamble Use Cases:**

1. **Application metadata:**
   ```tcl
   -preamble {set ::APP_VERSION "1.0.0"}
   ```

2. **Feature flags:**
   ```tcl
   -preamble {set ::ENABLE_DEBUG 0}
   ```

3. **Namespace initialization:**
   ```tcl
   -preamble {namespace eval ::myapp {variable data {}}}
   ```

4. **Environment setup:**
   ```tcl
   -preamble {
       set ::INSTALL_DIR [file dirname [info script]]
       lappend ::auto_path $::INSTALL_DIR/lib
   }
   ```

---

## Output Files

### File Format

Compiled bytecode files use the `.tbc` extension by default:

- **Format:** Binary file containing Tcl bytecode
- **Extension:** `.tbc` (Tcl ByteCode)
- **Compatibility:** Version-specific (Tcl 8.x or 9.x)

### File Size

Bytecode files are typically **larger** than source files:

- **Typical ratio:** 2-3x larger than `.tcl` source
- **Reason:** Includes compiled bytecode structures, metadata, and embedded information

**Example:**
```
script.tcl:  10 KB (source)
script.tbc:  25 KB (compiled bytecode)
```

### Output Location

**Default behavior:**
```tcl
compiler::compile /path/to/script.tcl
# Creates: /path/to/script.tbc (same directory as source)
```

**Custom location:**
```tcl
compiler::compile /path/to/script.tcl /output/dir/compiled.tbc
# Creates: /output/dir/compiled.tbc
```

### File Permissions

Output files inherit default permissions from the system:

- **Unix/Linux:** Typically `rw-r--r--` (644)
- **Windows:** Inherits directory permissions

---

## Error Handling

### Common Compilation Errors

#### 1. Source File Not Found

**Error:**
```
couldn't read file "script.tcl": no such file or directory
```

**Solution:**
```tcl
# Check file exists before compiling
if {![file exists $sourceFile]} {
    puts stderr "Error: Source file not found: $sourceFile"
    exit 1
}

compiler::compile $sourceFile
```

---

#### 2. Output Directory Doesn't Exist

**Error:**
```
couldn't open "output/script.tbc": no such file or directory
```

**Solution:**
```tcl
set outputFile "output/script.tbc"
set outputDir [file dirname $outputFile]

# Create directory if needed
file mkdir $outputDir

compiler::compile script.tcl $outputFile
```

---

#### 3. Syntax Errors in Source

**Error:**
```
syntax error in file "script.tcl" line 42: expected...
```

**Solution:**

The compiler reports syntax errors found during parsing. Fix the source file before recompiling.

```tcl
# Test source for syntax errors first
if {[catch {source script.tcl} err]} {
    puts stderr "Syntax error in source: $err"
    puts "Fix source file before compiling"
    exit 1
}

# Now compile
compiler::compile script.tcl
```

---

#### 4. Permission Denied

**Error:**
```
couldn't open "script.tbc": permission denied
```

**Solution:**
```bash
# Ensure write permissions on output directory
chmod u+w output/

# Or compile to a writable location
```

---

### Error Handling Pattern

Robust compilation with error handling:

```tcl
proc safe_compile {inputFile {outputFile ""}} {
    # Validate input
    if {![file exists $inputFile]} {
        return -code error "Input file not found: $inputFile"
    }

    if {![file readable $inputFile]} {
        return -code error "Input file not readable: $inputFile"
    }

    # Set output file
    if {$outputFile eq ""} {
        set outputFile "[file rootname $inputFile][compiler::getBytecodeExtension]"
    }

    # Ensure output directory exists
    set outputDir [file dirname $outputFile]
    if {![file isdirectory $outputDir]} {
        file mkdir $outputDir
    }

    # Compile
    if {[catch {
        compiler::compile $inputFile $outputFile
    } err]} {
        return -code error "Compilation failed: $err"
    }

    return $outputFile
}

# Usage
if {[catch {
    set output [safe_compile myapp.tcl]
    puts "Successfully compiled: $output"
} err]} {
    puts stderr "ERROR: $err"
    exit 1
}
```

---

## Performance Considerations

### Compilation Time

**Compilation is relatively fast:**

| Source Size | Compilation Time | Approximate Rate |
|-------------|------------------|------------------|
| 1,000 lines | ~0.1 seconds | ~10,000 lines/sec |
| 10,000 lines | ~0.5 seconds | ~20,000 lines/sec |
| 100,000 lines | ~3 seconds | ~33,000 lines/sec |

*Times vary by hardware and code complexity.*

### Runtime Loading Performance

**Benefits of bytecode:**

- **3-7x faster loading** compared to sourcing `.tcl` files
- **Reduced startup time** for applications with many procedures
- **No parsing overhead** - bytecode is ready to execute

**Runtime execution:**
- **Identical performance** to sourced scripts once loaded
- No runtime speed difference for computation

### When to Compile

**Good candidates for compilation:**

- Large applications (>1000 lines)
- Applications with many procedures
- Code that loads frequently
- Production deployments
- Commercial distributions

**Not worth compiling:**

- Tiny scripts (<100 lines)
- Frequently changing code during development
- Simple configuration files
- Scripts where source must be inspectable

---

## Security Considerations

### Important Security Notice

**Bytecode compilation provides OBFUSCATION, not cryptographic security.**

### What Bytecode Provides

✓ **Casual inspection protection** - Source not readable in text editor \
✓ **Basic IP protection** - Deters casual reverse engineering \
✓ **Distribution without source** - Ship applications without `.tcl` files

### What Bytecode Does NOT Provide

✗ **Cryptographic security** - Bytecode can be disassembled \
✗ **Complete source protection** - Determined attackers can reverse engineer \
✗ **Tamper resistance** - Bytecode files can be modified \
✗ **Secret storage** - Don't embed passwords/keys in bytecode

### Security Recommendations

1. **Use bytecode for commercial distribution** - Appropriate for protecting proprietary code
2. **Don't embed secrets** - Load credentials from external secure sources
3. **Combine with other protections:**
   - Code signing for integrity verification
   - License key validation systems
   - Online activation for high-value applications
4. **Keep runtime updated** - Update Tcl and tbcload when security patches are released

### Example: Secure Credential Handling

```tcl
# BAD: Embedding credentials in bytecode
proc connect {} {
    set password "secret123"  # Still visible in disassembly!
    database connect -password $password
}

# GOOD: Load credentials securely
proc connect {} {
    # Read from encrypted config, environment, or secure vault
    set password [read_encrypted_config "db_password"]
    database connect -password $password
}
```

---

## Best Practices

### 1. Maintain Source Files

**Always keep original `.tcl` source files:**

```tcl
# Good: Organize project with separate source and build directories
project/
  ├── src/          # Original .tcl files (version controlled)
  └── dist/         # Compiled .tbc files (generated)
```

**Never:**
- Delete source after compilation
- Version control only compiled `.tbc` files
- Rely on bytecode as sole backup

---

### 2. Version Control Strategy

**Include in version control:**
- ✓ All `.tcl` source files
- ✓ Build scripts
- ✓ Documentation

**Exclude from version control:**
- ✗ Compiled `.tbc` files (generated artifacts)
- ✗ Build output directories

**.gitignore example:**
```gitignore
# Ignore compiled bytecode
*.tbc
dist/
build/
```

---

### 3. Build Automation

**Create repeatable build processes:**

```tcl
# Makefile or build script
proc build_release {version} {
    puts "Building release $version..."

    # Clean previous build
    file delete -force dist
    file mkdir dist

    # Compile all sources
    foreach src [glob src/*.tcl] {
        set dest "dist/[file tail [file rootname $src]].tbc"
        compiler::compile $src $dest
    }

    # Create package
    create_package_index dist $version

    puts "Release $version built successfully"
}
```

---

### 4. Testing Strategy

**Test both source and bytecode:**

```tcl
# Run tests on source during development
tclsh tests/run_tests.tcl

# Compile
compiler::compile src/myapp.tcl dist/myapp.tbc

# Run tests on bytecode before release
package require tbcload
source dist/myapp.tbc
tclsh tests/run_tests.tcl
```

---

### 5. Documentation

**Document build requirements:**

```markdown
## Building from Source

Requirements:
- Tcl 8.6 or later
- tclcompiler package

Build:
```tcl
tclsh build.tcl release
```

Output: `dist/` directory with compiled .tbc files
```

---

## Troubleshooting

### Common Issues

#### 1. Package Not Found

**Error:**
```
can't find package tclcompiler
```

**Solutions:**
```tcl
# Check if installed
puts $auto_path

# Add installation directory
lappend auto_path /path/to/tclcompiler

# Verify package is available
package require tclcompiler
```

---

#### 2. Compilation Fails for Large Files

**Problem:** Compilation fails or is very slow for very large files

**Solutions:**
- Split large files into multiple smaller modules
- Increase system memory if needed
- Check for infinite loops or excessive recursion in source

---

#### 3. Bytecode Won't Load

**Error:**
```
couldn't load file "script.tbc": invalid or unsupported bytecode
```

**Causes:**
- Tcl version mismatch (compiled for 8.x, running on 9.x or vice versa)
- File corruption during transfer
- Wrong architecture (32-bit vs 64-bit)

**Solutions:**
```tcl
# Check version compatibility
puts "Compiler Tcl version: [compiler::getTclVer]"
puts "Runtime Tcl version: [info patchlevel]"

# Recompile with correct version
compiler::compile source.tcl output.tbc

# Verify file integrity
file stat output.tbc stat
puts "File size: $stat(size) bytes"
```

---

#### 4. Preamble Code Errors

**Problem:** Errors in preamble code cause compilation or runtime failures

**Solution:**
```tcl
# Test preamble code separately first
set preamble {
    set ::test_var 1
    namespace eval ::myapp {}
}

# Test it
eval $preamble

# Then use in compilation
compiler::compile -preamble $preamble input.tcl output.tbc
```

---

#### 5. Output File Not Created

**Problem:** Compilation reports success but file doesn't exist

**Debugging:**
```tcl
set outputFile "output.tbc"

# Verify output path
puts "Output: [file normalize $outputFile]"

# Check directory permissions
puts "Dir writable: [file writable [file dirname $outputFile]]"

# Try explicit path
compiler::compile input.tcl [file normalize $outputFile]

# Verify result
puts "Created: [file exists $outputFile]"
```

---

### Getting Help

If you encounter issues:

1. **Verify versions:**
   ```tcl
   puts "Tcl: [info patchlevel]"
   puts "tclcompiler: [package require tclcompiler]"
   puts "Compiler built for: [compiler::getTclVer]"
   ```

2. **Test with simple script:**
   ```tcl
   # test.tcl
   proc hello {} { puts "Hello" }

   # Compile
   compiler::compile test.tcl
   ```

3. **Check file paths:**
   ```tcl
   # Use absolute paths
   set abs_input [file normalize input.tcl]
   set abs_output [file normalize output.tbc]
   compiler::compile $abs_input $abs_output
   ```

---

## Building from Source

### Prerequisites

- **Tcl development headers** (`tcl-dev` or `tcl-devel`)
- **C compiler** (GCC, Clang, MSVC)
- **GNU Make**
- **autoconf** (if regenerating configure)
- **tbcload** (for shared header files)

### Build Instructions (Unix/Linux/macOS)

```bash
# Navigate to source directory
cd /path/to/tclcompiler

# Generate configure script (if needed)
autoconf

# Configure
./configure --with-tcl=/usr/lib/tcl8.6

# Alternative: auto-detect
./configure

# Build
make

# Test (optional)
make test

# Install (requires sudo/root)
sudo make install
```

### Build Instructions (Windows with MinGW/MSYS2)

```bash
# Open MinGW/MSYS2 shell
cd /path/to/tclcompiler

# Configure
./configure --with-tcl=/c/Tcl/lib

# Build
make

# Install
make install
```

### Build Instructions (Windows with MSVC)

```cmd
REM Open Visual Studio Command Prompt
cd C:\path\to\tclcompiler\win

REM Edit makefile.vc to set paths
REM Build
nmake -f makefile.vc

REM Install
nmake -f makefile.vc install
```

### Configuration Options

```bash
# Specify Tcl location
./configure --with-tcl=/path/to/tclConfig.sh

# Specify tbcload headers location
./configure --with-tbcload=/path/to/tbcload

# Enable debugging symbols
./configure --enable-symbols

# Build in separate directory
mkdir build && cd build
../configure
make
```

### Verification

```tcl
# After installation
package require tclcompiler
puts "tclcompiler [package present tclcompiler] installed"
puts "Built for Tcl [compiler::getTclVer]"

# Test compilation
compiler::compile test.tcl
puts "Compilation test successful"
```

---

## Platform Support

The tclcompiler package supports all platforms where Tcl is available:

### Tested Platforms

- **Windows** - Windows 7/8/10/11 (32-bit and 64-bit)
- **Linux** - All major distributions
- **macOS** - 10.9+ (Intel and Apple Silicon)
- **BSD** - FreeBSD, OpenBSD, NetBSD
- **Unix** - Solaris, AIX, HP-UX

### Platform-Specific Notes

**Windows:**
- Requires `tclcompiler.dll` matching Tcl version
- Visual Studio or MinGW builds available
- Compatible with ActiveState, Magicsplat Tcl distributions

**Linux:**
- Shared library: `libtclcompiler.so`
- Available in some distribution repositories
- Build from source for custom Tcl installations

**macOS:**
- Universal binaries support Intel and ARM
- Works with system Tcl or custom builds
- Shared library: `libtclcompiler.dylib`

---

## License

**BSD 3-Clause License**

Copyright (c) 1999-2000 Ajuba Solutions \
Copyright (c) 2018 ActiveState Software Inc.

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

3. Neither the name of the copyright holder nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

---

## Version History

- **2.0a0** (Current) - Tcl 9.0 support release
  - Added Tcl 9.0 compatibility
  - Modern bytecode format generation
  - Compatible with tbcload 2.0+
  - Modernized package structure

- **1.6** (2006-09-11)
  - Added Tcl 8.5 support for compiled switch statements (JumpTableInfo)

- **1.5** (2002-09-30)
  - Format version adapts to match interpreter version

- **Earlier versions** - See ChangeLog for historical releases from TclPro era

---

## Additional Resources

### Official Documentation

- [TclPro User's Guide - Chapter 6](https://www.tcl-lang.org/software/tclpro/doc/TclProUsersGuide14.pdf) - Comprehensive compiler guide
- [tclcompiler GitHub Repository](https://github.com/tcltk-depot/tclcompiler) - Source code and issues

### Related Packages

- **tbcload** - Required companion package for loading compiled bytecode
- **TclDevKit** - Commercial suite with enhanced compiler features
- **procomp** - Legacy TclPro compiler (superseded by tclcompiler)

### Community Resources

- [Tcl Wiki - tclcompiler](https://wiki.tcl-lang.org/page/tclcompiler)
- [Tcl Community Forums](https://groups.google.com/g/comp.lang.tcl)
- [Stack Overflow - Tcl tag](https://stackoverflow.com/questions/tagged/tcl)

### Related Topics

- Tcl bytecode internals
- Application deployment and distribution
- Code protection and obfuscation
- Tcl Extension Architecture (TEA)

---

## Quick Reference

### Basic Commands

```tcl
# Load package
package require tclcompiler

# Compile with automatic output
compiler::compile script.tcl              # Creates script.tbc

# Compile with explicit output
compiler::compile script.tcl output.tbc

# Compile with preamble
compiler::compile -preamble {set ::ver "1.0"} script.tcl output.tbc

# Get bytecode extension
set ext [compiler::getBytecodeExtension]  # Returns ".tbc"

# Get compiler Tcl version
set ver [compiler::getTclVer]             # Returns "8.6" or "9.0"
```

### Common Patterns

```tcl
# Batch compile directory
foreach file [glob src/*.tcl] {
    set base [file rootname [file tail $file]]
    compiler::compile $file "dist/${base}.tbc"
}

# Version check before compiling
set compVer [compiler::getTclVer]
set rtVer [lindex [split [info patchlevel] .] 0]
if {$compVer != $rtVer} {
    error "Version mismatch: compiler=$compVer runtime=$rtVer"
}
compiler::compile script.tcl

# Safe compilation with error handling
if {[catch {
    compiler::compile input.tcl output.tbc
} err]} {
    puts stderr "Compilation failed: $err"
    exit 1
}
```

### Remember

- **Version matching critical** - Compile for the target Tcl version (8.x or 9.x)
- **Keep source files** - Always maintain original `.tcl` sources
- **Test bytecode** - Verify compiled output works correctly
- **Bytecode is obfuscation** - Not cryptographic protection
- **Requires tbcload** - Runtime needs tbcload to execute `.tbc` files

---

**Compile your Tcl applications for faster loading and source protection!**
