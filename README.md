# Geode Docs

### [Visit the Docs site](https://docs.geode-sdk.org)

This is the source code for Geode's docs, containg all the hand-written tutorials.

Class & function documentation is built automatically from [the Geode source code](https://github.com/geode-sdk/geode).

## Building

The docs are built using [Flash](https://github.com/hjfod/flash). To build the docs, you need Flash, along with [CMake](https://cmake.org/install/) and [Clang](https://clang.llvm.org/).

After building Flash from source using Cargo or installing the latest release, you can build the docs with the following command:

```
flash -i <path/to/geode> -o <relative_output_dir> --overwrite
```

Afterwards, start up a local HTTP server in the folder where you ran Flash.

You should run Flash in the directory you want to build the docs in and use `-o .`, or run it in the parent directory and do `-o <output_dir_name>`.
