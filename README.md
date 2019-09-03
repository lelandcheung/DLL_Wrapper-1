# Forked For
Updated to VS2019 and fixed an issue that was causing repition of function parameters in the creation of the proxy templates.


# DLL_Wrapper

A program that generates code to implement a DLL Proxy.

Also known as DLL Reflection or DLL Redirection.

## About

DLL Proxying is a technique in which an attacker replaces a DLL with a Trojan
version, renaming the original rather than deleting it. This Trojan DLL
implements functions which the attacker wishes to intercept or modify, while
forwarding all other functions to the original DLL.  The attacker can thus Man
in the Middle the functions they’re interested in, and forward the rest to the
original DLL, minimizing the amount of work needed while ensuring functionality
is not reduced or broken.

The entire attack is conducted in a six-step process:
  1.	Analyze original DLL
  2.	Identify functions to modify
  3.	Implement modified functions in Trojan DLL
  4.	Forward all other functions to the original DLL
  5.	Rename original DLL
  6.	Place Trojan DLL

This project supports x86 and x64, and has been tested on Windows 10, but should
work with Windows 7 and up.

# Goals

* Full x86 and x64 Support
* Support for Exports as Ordinals
* Support for Forward Exports as Ordinals
* Generate stub code for intercepted functions that load the original target
DLL, as well as a handle to the original function being intercepted. A printf
call made to prove interception (as an example).
* Generate Assembly (MASM) stub code to jump into original function being
intercepted. Note: Inline assembly isn't supported in x64 with Visual Studios.

## Overview

While the entire attack is a six-step process, this process can be grouped into
two phases:

1.	Creation of the Trojan DLL
2.	Implementation of the Trojan DLL.

In the first phase, the Trojan DLL must be coded, with intercepted functions
implemented and exported. All remaining functions must make use of the PE
formats Forward Exports to export to the original DLL.

In the second phase,
write permissions will be required at the target DLLs location to rename the
original DLL, and write the Trojan in its place.

This project focuses on the first phase of the process. It will generate the
code to intercept the desired functions, as well as stub code to then call the
original implementation of that function. The linker commands required to
implement Forward Exports for non-implemented functions will also be generated.

Generated code can then be imported into a Visual Studio DLL Project, and
compiled into a DLL.

Generated code must be able to intercept by name or ordinal value, forward by name
or ordinal value, but must also handle any forward exports the target DLL may poses.

The following must thus be handled:

**Intercepts**
  - Intercept by name
  - Intercept by ordinal
  
**Forward Exports**
  - Forward by name to a function exported by name
  - Forward by ordinal to a function exported by ordinal
  - Forward by name to a function exported by ordinal
  - Forward by ordinal to a function exported by name

## Build

Open DLL_Wrapper.sln with Visual Studio and build Debug or Release, x86 or x64
depending on needs.

# Usage

This program uses a CLI, and must be run from cmd.exe. No GUI yet.

One argument is required by the program, the path to a configuration file.

```
C:\Users\User> DLL_Wrapper.exe configuration.xml
```

There is a configuration file *configuration.xml* at
*DLL_Wrapper/DLL_Wrapper/configuration.xml* with a mock example.

To clarify:
  -	\<target>: Path to the DLL you’re proxying
  -	\<rename_target>: The name that will be given to the original DLL after it
  is renamed
  -	\<output_dir>: The directory to output generated source code
  -	\<intercepts>: List of functions to intercept
    -	\<function>: Function to intercept
        - \<name>: Name of the function to intercept (can be an ordinal)
      	- \<paramaters>: (optional) list of parameters for the function
          - \<param>: Function parameter. Attribute type is required.

To intercept by ordinal the configuration file requires the following format for the name:
```"ord"``` + ```ordinal_value```. For example, to intercept ordinal 5: ```ord5```.

Generated stub code to implement a Proxy DLL will be output to the <outuput_dir>. 

A subtle but important implementation to note; this stub code will produce both a *DEF file*
and make use of *VS Linker Directives* when exporting functions:
  - **DEF file**: used to export intercepted functions with a specific ordinal value. It is
  not used for Forward Exports due to a linking verification done by the linker which would
  prevent the build of the Proxy DLL.
  - **VS Linker Directives**: placed in a header file ```forwards.h```, these directives allow
  us to create Forward Export entries in the Exports Directory of the DLL without the linker
  verification.
  
I am not sure why a linking verification is done when specifying a Forward Export in a DEF file,
nor why it is NOT performed when using VS Linker Directives.

# Example

[COMING SOON]

An example of a DLL which is proxied and used by a victim application can be found here:  https://github.com/kevinalmansa/DLL_Wrapper_Example

# License

Licensed under the MIT License. Please see LICENSE for details.
