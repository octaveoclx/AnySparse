# AnySparse

A streamlined, directly compilable port of **clSPARSE** with a focus on simplicity and broad OpenCL device support.


## Recent Updates (2026‑05‑21)

- 0-indexing for MMGenerateCOOFromFile/clsparseSCsrMatrixfromFile.
- Dense B and C with different row/column major in clsparseScsrmm.
- Double-precision support for sparse matrix multiplication thanks to undergraduate student Zhipeng Liu @ College of Computer Science and Technology, GZU as part of his graduation project under my supervision.

## Updates (2026‑05‑19)

- Overhauled Makefiles to support the latest OpenCL libraries and tools.
- Removed all legacy kernel compilation steps; kernels are now shipped as plain `.cl` files within the source tree.
- General code cleanup and variable initialization fixes.


## Overview

This project, maintained by **Prof. Jinchuan Tang** ([jctang@gzu.edu.cn](mailto:jctang@gzu.edu.cn)), is a derivative of the original clSPARSE library. Its primary goal is to eliminate the complexities of the original build system, offering a straightforward "clone and compile" experience.

The library supports any OpenCL 1.2+ capable device and is designed for users who need sparse linear algebra operations without the overhead of managing complex build dependencies.

## Supported Operations

- **Sparse BLAS Level 1**: Operations like `axpy`, `dot`, norms.
- **Sparse BLAS Level 2**: Sparse Matrix-Vector multiplication (SpMV).
- **Sparse BLAS Level 3**: Sparse Matrix-Matrix multiplication (SpMM).
- **Solvers**: Triangular solvers (SpTrSV, SpTrSM).
- **Formats**: Conversion between CSR, COO, ELL, and other sparse storage formats.
- **Preconditioners**: Basic preconditioners like diagonal scaling.

## Supported Precisions

The library currently provides operations for following data type.

- **Single (float)**: Fully supported.
- **Double (double)**: Enabled.

## Key Improvements over Original clSPARSE

- **Simplified Build**: Eliminates the need for intermediate kernel generation. Just edit the provided sample configuration file to set your OpenCL and library paths, then run `make`.
- **Sample Configuration File**: A working example is included to guide you through setting up your own build.
- **Cross-Platform Ready**: Tested and verified on **Windows 11**, **Ubuntu 26.04**, and **macOS 26.5**.
- **All Sources Included**: The repository contains all necessary source and kernel files.


---

## Integrating AnySparse into Your Own Project

AnySparse is designed to be compiled **together with your own source code**, not as a separate library. You simply add the AnySparse source tree (the folder you get after extracting the project) to your project and let the Makefile compile everything in one go.

### Directory Structure of AnySparse (typical)

When you extract the AnySparse distribution, you get a root folder (named `anysparse` or similar) with the following internal layout:

    anysparse/                        # project root directory
    ├── include/                      # public headers
    │   ├── clSPARSE.h
    │   ├── clSPARSE-1x.h
    │   └── ...
    ├── library/                      # all internal sources and private headers
    │   ├── blas1/                    # Level 1 sparse routines
    │   ├── blas2/                    # Level 2 sparse routines
    │   ├── blas3/                    # Level 3 sparse routines
    │   ├── include/                  # internal headers (e.g., clSPARSE-private.hpp)
    │   ├── internal/                 # core infrastructure (kernel cache, control, etc.)
    │   ├── io/                       # input/output utilities
    │   ├── kernels/                  # OpenCL kernel files (.cl)
    │   ├── solvers/                  # iterative solvers
    │   ├── transform/                # format conversion routines
    │   ├── clsparse-init.cpp
    │   └── ...
    └── clsparseTimer/                # optional timing helpers (if needed)

### Example Project Structure

Assume you have extracted the AnySparse distribution into a folder named `anysparse` inside your own project:

    your_project/
    ├── Makefile
    ├── src/                          # your own source code
    │   ├── main.c
    │   └── utils/
    │       └── helper.c
    ├── include/                      # your own headers (optional)
    └── anysparse/                    # AnySparse source tree
        ├── include/
        ├── library/
        └── clsparseTimer/

### Makefile Example (compile everything together)

    # ============================================================================
    # Project integrating AnySparse source directly
    # ============================================================================
    
    # Target executable
    TARGET = my_sparse_app
    
    # Path to the AnySparse source root (folder extracted from the distribution)
    ANYSPARSE_DIR = ./anysparse
    
    # Compiler settings
    CC = gcc
    CXX = g++
    CFLAGS = -O2 -Wall -fPIC
    CXXFLAGS = $(CFLAGS) -std=c++11
    
    # OpenCL and math libraries
    LDFLAGS = -lOpenCL -lm
    
    # Include paths: your own headers + AnySparse public + AnySparse private
    INC_DIRS = -Iinclude \
               -I$(ANYSPARSE_DIR)/include \
               -I$(ANYSPARSE_DIR)/library/include
    
    # Find all source files: your own + AnySparse sources
    YOUR_SRC_DIRS = src
    YOUR_SOURCES = $(shell find $(YOUR_SRC_DIRS) -type f \( -name "*.c" -o -name "*.cc" -o -name "*.cpp" \))
    
    # AnySparse sources: everything under library/ (and clsparseTimer if needed)
    # Exclude hidden directories, build artifacts, etc.
    ANYSPARSE_SOURCES = $(shell find $(ANYSPARSE_DIR) -type f \( -name "*.c" -o -name "*.cc" -o -name "*.cpp" \) \
                         -not -path "*/\.*" -not -path "*/build/*")
    
    ALL_SOURCES = $(YOUR_SOURCES) $(ANYSPARSE_SOURCES)
    
    # Convert to object files (preserve relative paths)
    OBJECTS = $(patsubst %.c,%.o,$(patsubst %.cc,%.o,$(patsubst %.cpp,%.o,$(ALL_SOURCES))))
    
    # Helper to create directory for object files
    define ensure_dir
        @mkdir -p $(dir $@)
    endef
    
    all: $(TARGET)
    
    $(TARGET): $(OBJECTS)
        $(CXX) $^ $(LDFLAGS) -o $@
    
    # Rule for C sources
    %.o: %.c
        $(ensure_dir)
        $(CC) $(CFLAGS) $(INC_DIRS) -c $< -o $@
    
    # Rule for C++ sources (.cc)
    %.o: %.cc
        $(ensure_dir)
        $(CXX) $(CXXFLAGS) $(INC_DIRS) -c $< -o $@
    
    # Rule for C++ sources (.cpp)
    %.o: %.cpp
        $(ensure_dir)
        $(CXX) $(CXXFLAGS) $(INC_DIRS) -c $< -o $@
    
    clean:
        @find $(YOUR_SRC_DIRS) $(ANYSPARSE_DIR) -name "*.o" -delete
        @rm -f $(TARGET)
    
    .PHONY: all clean

### Important Notes

- The Makefile compiles **all** source files from `src/` and `anysparse/` together. No separate static or shared library is built.
- The two include directories `-I$(ANYSPARSE_DIR)/include` and `-I$(ANYSPARSE_DIR)/library/include` are both necessary. The first gives public headers (e.g., `clSPARSE.h`), the second gives internal headers (e.g., `clSPARSE-private.hpp`).
- The `find` command automatically picks up `.cpp` and `.c` files inside `library/` and any subdirectories (blas1, blas2, transform, solvers, etc.). It also includes `clsparseTimer` if present; you can exclude it by adding `-not -path "*/clsparseTimer/*"` if desired.
- You must have an OpenCL implementation (headers + runtime library) installed.
- On Windows with MingW64, use `ming32-make` and ensure OpenCL libraries are in your `PATH` or `LIBRARY_PATH`.

### Using AnySparse in Your Source Code

In your C/C++ files, include the main header:

    #include <clSPARSE.h>

Then initialize an OpenCL context, create a sparse matrix, and call AnySparse functions. For example:

    cl_context context = ...;
    cl_command_queue queue = ...;
    
    clsparseCsrMatrix A;
    clsparseCsrMatrixInit(&A);
    A.values = ...;   // allocate device memory
    A.colIndices = ...;
    A.rowOffsets = ...;
    A.num_rows = rows;
    A.num_cols = cols;
    A.num_nonzeros = nnz;
    
    clsparseStatus status = clsparseScsrmv(&alpha, A, x, &beta, y, queue, &control);
    
    clsparseCsrMatrixFree(&A, &control);

### Runtime Kernel Loading (Optional)

If you prefer to load OpenCL kernels from external `.cl` files (for experimentation or debugging), set the environment variable `CLSPARSE_KERNEL_PATH` to the directory containing the `.cl` kernel files (usually `anysparse/library/kernels/`). Otherwise, the built‑in kernels (compiled into the library via strings) are used by default.

---

## License

Refer to the original clSPARSE license (Apache 2.0 / BSD‑style). See `COPYRIGHT` in the AnySparse distribution for details.

## Contact

Prof. Jinchuan Tang – [jctang@gzu.edu.cn](mailto:jctang@gzu.edu.cn)  
Issues and pull requests are welcome on the GitHub repository.
