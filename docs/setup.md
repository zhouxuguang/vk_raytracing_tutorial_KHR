# Setup Guide

This guide will help you set up a working copy of the foundation sample for the Vulkan Ray Tracing Tutorial.

## Important: Create a Working Copy

Before starting this tutorial, **create a copy of the `01_foundation` directory** to work with:

```bash
# Navigate to the raytrace_tutorial directory
cd raytrace_tutorial
``` 

Create a working copy of the foundation sample

```bash
# Linux
cp -r 01_foundation 01_foundation_copy
# Windows
xcopy 01_foundation 01_foundation_copy /S /E
``` 

## Why Create a Copy?

- **Preserves Original**: Keeps `01_foundation` intact for reference
- **Easy Comparison**: Compare your progress with the original implementation
- **Clean Starting Point**: Fresh copy for each phase
- **Easy Rollback**: Can restart from clean state if needed
- **Reference Material**: Original serves as a working example


## CMakeLists.txt Modification

You need to add the new project to the main CMakeLists.txt. Insert this line after the original foundation project:

```cmake
add_subdirectory(raytrace_tutorial/01_foundation)
add_subdirectory(raytrace_tutorial/01_foundation_copy)
add_subdirectory(raytrace_tutorial/02_basic)
```

## Working Directory Structure

After creating the copy, your directory should look like this:

```
raytrace_tutorial/
├── 01_foundation/                    # Original (don't modify)
│   ├── 01_foundation.cpp
│   ├── shaders/
│   │   ├── foundation.slang
│   │   └── shaderio.h
│   ├── CMakeLists.txt
│   └── README.md
├── 01_foundation_copy/               # Your working copy
│   ├── 01_foundation.cpp             # Main file to modify
│   ├── shaders/                      # Will add rtbasic.slang
│   │   ├── foundation.slang         
│   │   └── shaderio.h
│   ├── CMakeLists.txt
│   └── README.md
├── 02_basic/                         # Reference implementation
│   ├── 02_basic.cpp
│   ├── shaders/
│   │   ├── rtbasic.slang
│   │   └── shaderio.h
│   └── CMakeLists.txt
└── docs/
    └── index.md   # Main tutorial (includes setup instructions)
```

## Build Verification

Since this is a unified build system, you only need to rebuild from the root directory:

```bash
# From the project root directory
cmake -B build -S .
cmake --build build -j 8
```

This will build all projects including both:

- `01_foundation` (original)
- `01_foundation_copy` (your working copy)

Both should produce identical results initially.
