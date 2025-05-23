# OpenSSL provider using Cryptography API: Next Generation

Basic OpenSSL provider implementation that uses Windows Cryptography API: Next generation. It is primarily
meant to be a stepping stone for anybody wanting to implement their own provider. The functionality of
this provider is in greater detail in the section `This providers functionality`. 

Any feedback or pull requests are welcome.

The code is licensed under the MIT license, in case anybody would like to have even more permissive license,
create an issue, and I'm going to try and find a solution.

## This providers functionality
This provider allows retrieval of certificates and their associated RSA keys (other types will be skipped) stored in Windows system stores. Associated
keys can be used to sign digests with SHA2-256, 386 and 512 using OpenSSL API (with the work being done by CNG so
non-exportable private keys can be used as well).
## Usage of this provider
To load this provider use the name `cng_provider` in either the `-provider` command line argument where supported or
in `OSSL_PROVIDER_load()`. To maintain full functionality od OpenSSL, also load the `default` provider.

This provider requires a URI with `cng://` schema. After the schema comes the Windows system store name. Currently
supported are:
```
cng://CA
cng://MY
cng://ROOT
```

after the store name an optional store location can be specified by using the `@` symbol. For example: `cng://MY@currentuser`

Currently supported are:
```
currentmachine
localmachine
```

## Example commandline usage
```
openssl s_client -provider cng_provider -provider default -connect certificate-required-website.example.com:443 -cert cng://MY@currentuser
```
## Example in-code usage
An example of how one might write code with this provider, that loads a specific certificate from the Windows store,
can be found in the `client` folder.

## Prerequisites for usage: 
  * OpenSSL 3.0.0+ (should be 3.2.0 compatible as well)
  * Windows with CNG support

## Prerequisites for compilation:
  * Prerequisites for usage
  * Visual Studio with C++ compilation packages
  * Strawberry Perl
  * NASM

Add NASM and Strawberry Perl to `PATH`.

```
cpan -i Text::Template
cpan -i Test::More
```

Now you should be ready for the next step.

## Compilation
Make sure you have the all the `prerequisities for compilation`.

It is assumed, that the final product should be a `x64` dynamically loadable provider. So OpenSSL and the provider are
both compiled in `x64` mode. See section `x86 compilation` for other architectures.

### Compilation of OpenSSL:
```
"C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvarsall.bat" amd64
cd "C:\path\to\root\of\openssl\repo"
perl Configure VC-WIN64A -d no-shared enable-trace no-engine
nmake
nmake test
nmake install_ssldirs install_sw
```
Optionally also run `nmake install install_docs`.

### Compilation of this project
```
cmake -S . -B ./custom-build-directory
cmake --build ./custom-build-directory --target cng_provider
cmake --build ./custom-build-directory --target install
cmake --build ./custom-build-directory --target client
```

The install path is set in CMakeLists.txt with `CMAKE_INSTALL_PREFIX` variable.

### x86 compilation
Make sure you compile OpenSSL in `x86` mode and have it installed. Use `amd64_x86` for `vcvarsall.bat` and `VC-WIN32` for `perl Configure`.
Change the appropriate install directory and OpenSSL location in `CMakeLists.txt`.

### Common problems
  * You are trying to comile debug version and CMake cannot find your OpenSSL binaries: Easy solution is to name them `libcryptod.lib` and `libssld.lib`.
  * Provider cannot be installed/loaded: Check permissions on your `ossl-modules` folder and files in there, your user must be able to read (and/or write) there.
  * I changed the architecture and now it does not compile: Clean your caches and temporary files. Make sure you have the appropriate (`x64`/`x86`) OpenSSL version.
  * I cannot compile OpenSSL in `x86` mode: You have to change two arguments during OpenSSL compilation and then two other in `CMakeLists.txt`.
