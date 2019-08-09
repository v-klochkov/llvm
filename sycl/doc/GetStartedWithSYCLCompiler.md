# Overview
The SYCL* Compiler compiles C++\-based SYCL source files with code for both CPU
and a wide range of compute accelerators. The compiler uses Khronos*
OpenCL&trade; API to offload computations to accelerators.

---
---
---

# Before You Begin

### Get `OpenCL runtime` for CPU and/or GPU:

   a. OpenCL runtime for GPU: follow instructions on
[github.com/intel/compute-runtime/releases](https://github.com/intel/compute-runtime/releases)
to install.

   b. Experimental Intel&reg; CPU Runtime for OpenCL&trade; Applications with
SYCL support: follow the instructions under
[SYCL* Compiler and Runtimes](https://github.com/intel/llvm/releases/tag/2019-07)

### Get the required tools:

   a. `git` - for downloading the sources
   
   b. `cmake` - for building the compiler and tools (Get it at: http://www.cmake.org/download)
      
   c. `python` - for building the compiler and running tests (Get it at: https://www.python.org/downloads/release/python-2716/ )

   d. `Visual Studio 2017 or later` (Windows only)


---
---
---

# Build the SYCL compiler and runtime

Download the LLVM repository with SYCL support and required tools to your local machine folder, e.g. $WORK_DIR. For simplicity it is assumed below that environment variable WORK_DIR contains path to an existing folder.

## Set some `environment variables`. For example:
### Linux:
```bash
export WORK_DIR=/export/home/work_dir
export SYCL_HOME=$WORK_DIR/sycl_workspace
```
### Windows:
Open a developer command prompt using one of tho methods:
- Click start menu and search forthe command prompt. So, for MSVC-2017 it is '`x64 Native Tools Command Prompt for VS 2017`'
- run 'cmd' and then '`"C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build\vcvarsall.bat" x64`'
```bash
set WORK_DIR=%USERPROFILE%\work_dir
set SYCL_HOME=%WORK_DIR%\sycl_workspace
```
---
## Get `OpenCL-Headers`
### Linux:
```bash
cd $WORK_DIR
git clone https://github.com/KhronosGroup/OpenCL-Headers
export OPENCL_HEADERS=$WORK_DIR/OpenCL-Headers
```

### Windows:
```bash
cd %WORK_DIR%
git clone https://github.com/KhronosGroup/OpenCL-Headers
set OPENCL_HEADERS=%WORK_DIR%\OpenCL-Headers
```

---
## Get `OpenCL-ICD-Loader`
You can also find the most recent instructions for this component at https://github.com/KhronosGroup/OpenCL-ICD-Loader
### Linux:
```bash
cd $WORK_DIR
git clone https://github.com/KhronosGroup/OpenCL-ICD-Loader
cd OpenCL-ICD-Loader
mkdir build
cd build
cmake ..
make C_INCLUDE_PATH=$OPENCL_HEADERS
export ICD_LIB=$WORK_DIR/OpenCL-ICD-Loader/build/libOpenCL.so
```

### Windows:
```bash
cd %WORK_DIR%
git clone https://github.com/KhronosGroup/OpenCL-ICD-Loader
cd OpenCL-ICD-Loader
mkdir build
cd build
set INCLUDE=%OPENCL_HEADERS%;%INCLUDE%
cmake -G "Ninja" -DOPENCL_ICD_LOADER_REQUIRE_WDK= ..
ninja
set ICD_LIB=%WORK_DIR%\OpenCL-ICD-Loader\build\OpenCL.lib
```
---

## Checkout and build the `Compiler with SYCL runtime`
The compiler is downloaded and built in `$WORK_DIR`/`$SYCL_HOME` directory (assuming SYCL_HOME environment var is set properly) 
### Linux:
```bash
mkdir $SYCL_HOME
cd $SYCL_HOME
git clone https://github.com/intel/llvm -b sycl
mkdir $SYCL_HOME/build
cd $SYCL_HOME/build

cmake -DCMAKE_BUILD_TYPE=Release \
-DOpenCL_INCLUDE_DIR=$OPENCL_HEADERS -DOpenCL_LIBRARY=$ICD_LIB \
-DLLVM_EXTERNAL_PROJECTS="llvm-spirv;sycl" \
-DLLVM_ENABLE_PROJECTS="clang;llvm-spirv;sycl" \
-DLLVM_EXTERNAL_SYCL_SOURCE_DIR=$SYCL_HOME/llvm/sycl \
-DLLVM_EXTERNAL_LLVM_SPIRV_SOURCE_DIR=$SYCL_HOME/llvm/llvm-spirv \
$SYCL_HOME/llvm/llvm

make -j`nproc` sycl-toolchain
```
### Windows:
```bash
mkdir %SYCL_HOME%
cd %SYCL_HOME%
git clone https://github.com/intel/llvm -b sycl
mkdir %SYCL_HOME%\build
cd %SYCL_HOME%\build

cmake -G "Ninja" -DCMAKE_BUILD_TYPE=Release -DOpenCL_INCLUDE_DIR="%OPENCL_HEADERS%" -DOpenCL_LIBRARY="%ICD_LIB%" -DLLVM_EXTERNAL_PROJECTS="llvm-spirv;sycl" -DLLVM_ENABLE_PROJECTS="clang;llvm-spirv;sycl" -DLLVM_EXTERNAL_SYCL_SOURCE_DIR="%SYCL_HOME%\llvm\sycl" -DLLVM_EXTERNAL_LLVM_SPIRV_SOURCE_DIR="%SYCL_HOME%\llvm\llvm-spirv" -DLLVM_TARGETS_TO_BUILD="X86" -DCMAKE_C_COMPILER=cl -DCMAKE_CXX_COMPILER=cl -DCMAKE_C_FLAGS="/GS" -DCMAKE_CXX_FLAGS="/GS" -DCMAKE_EXE_LINKER_FLAGS="/NXCompat /DynamicBase" -DCMAKE_SHARED_LINKER_FLAGS="/NXCompat /DynamicBase" "%SYCL_HOME%\llvm\llvm"

ninja sycl-toolchain
```

After the build completed, the SYCL compiler/include/libraries can be found
in `$SYCL_HOME/build` directory.

---

## Build the SYCL runtime with libc++ library.

There is experimental support for building and linking SYCL runtime with
libc++ library instead of libstdc++. To enable it the following cmake options
should be used.
### Linux:
```
-DSYCL_USE_LIBCXX=ON \
-DSYCL_LIBCXX_INCLUDE_PATH=<path to libc++ headers> \
-DSYCL_LIBCXX_LIBRARY_PATH=<path to libc++ and libc++abi libraries>
```

# Test the SYCL compiler and runtime

Run LIT testing using the command below after building SYCL compiler and runtime.
Use one of these commands:
### Linux:
```bash
make -j`nproc` check-all
make -j`nproc` check-llvm
make -j`nproc` check-llvm-spirv
make -j`nproc` check-clang
make -j`nproc` check-sycl
```
### Windows:
```bash
ninja check-all
ninja check-llvm
ninja check-llvm-spirv
ninja check-clang
ninja check-sycl
```
If no OpenCL GPU/CPU runtimes are available, the corresponding LIT tests are skipped

---
---
---
# Creating a simple SYCL program

A simple SYCL program consists of following parts:
1. Header section
2. Allocating buffer for data
3. Creating SYCL queue
4. Submitting command group to SYCL queue which includes the kernel
5. Wait for the queue to complete the work
6. Use buffer accessor to retrieve the result on the device and verify the data
7. The end

Creating a file `simple-sycl-app.cpp` with the following C++ SYCL code in it:

```c++
#include <CL/sycl.hpp>

int main() {
  // Creating buffer of 4 ints to be used inside the kernel code
  cl::sycl::buffer<cl::sycl::cl_int, 1> Buffer(4);

  // Creating SYCL queue
  cl::sycl::queue Queue;

  // Size of index space for kernel
  cl::sycl::range<1> NumOfWorkItems{Buffer.get_count()};

  // Submitting command group(work) to queue
  Queue.submit([&](cl::sycl::handler &cgh) {
    // Getting write only access to the buffer on a device
    auto Accessor = Buffer.get_access<cl::sycl::access::mode::write>(cgh);
    // Executing kernel
    cgh.parallel_for<class FillBuffer>(
        NumOfWorkItems, [=](cl::sycl::id<1> WIid) {
          // Fill buffer with indexes
          Accessor[WIid] = (cl::sycl::cl_int)WIid.get(0);
        });
  });

  // Getting read only access to the buffer on the host.
  // Implicit barrier waiting for queue to complete the work.
  const auto HostAccessor = Buffer.get_access<cl::sycl::access::mode::read>();

  // Check the results
  bool MismatchFound = false;
  for (size_t I = 0; I < Buffer.get_count(); ++I) {
    if (HostAccessor[I] != I) {
      std::cout << "The result is incorrect for element: " << I
                << " , expected: " << I << " , got: " << HostAccessor[I]
                << std::endl;
      MismatchFound = true;
    }
  }

  if (!MismatchFound) {
    std::cout << "The results are correct!" << std::endl;
  }

  return MismatchFound;
}

```

---
---
---
# Build and Test a simple SYCL program

To build simple-sycl-app put `bin` and `lib` to PATHs and run following command:
### Linux:
   ```bash
   export PATH=$SYCL_HOME/build/bin:$PATH
   export LD_LIBRARY_PATH=$SYCL_HOME/build/lib:$LD_LIBRARY_PATH
   ```
### Windows:
   ```bash
   set PATH=%SYCL_HOME%\build\bin;%PATH%
   set LIB=%SYCL_HOME%\build\lib;%LIB%
   ```

### Linux & Windows:
   ```bash
   clang++ -fsycl simple-sycl-app.cpp -o simple-sycl-app.exe -lOpenCL
   ```

This `simple-sycl-app.exe` application doesn't specify SYCL device for execution,
so SYCL runtime will first try to execute on OpenCL GPU device first, if OpenCL
GPU device is not found, it will try to run OpenCL CPU device; and if OpenCL
CPU device is also not available, SYCL runtime will run on SYCL host device.

### Linux & Windows:
   ```bash
   ./simple-sycl-app.exe
   The results are correct!
   ```


NOTE: SYCL developer can specify SYCL device for execution using device
selectors (e.g. `cl::sycl::cpu_selector`, `cl::sycl::gpu_selector`) as
explained in following section [Code the program for a specific
GPU](#code-the-program-for-a-specific-gpu).

---
---
---
# Code the program for a specific GPU

To specify OpenCL device SYCL provides the abstract `cl::sycl::device_selector`
class which the can be used to define how the runtime should select the best
device.

The method `cl::sycl::device_selector::operator()` of the SYCL
`cl::sycl::device_selector` is an abstract member function which takes a
reference to a SYCL device and returns an integer score. This abstract member
function can be implemented in a derived class to provide a logic for selecting
a SYCL device. SYCL runtime uses the device for with the highest score is
returned. Such object can be passed to `cl::sycl::queue` and `cl::sycl::device`
constructors.

The example below illustrates how to use `cl::sycl::device_selector` to create
device and queue objects bound to Intel GPU device:

```c++
#include <CL/sycl.hpp>

int main() {
  class NEOGPUDeviceSelector : public cl::sycl::device_selector {
  public:
    int operator()(const cl::sycl::device &Device) const override {
      using namespace cl::sycl::info;

      const std::string DeviceName = Device.get_info<device::name>();
      const std::string DeviceVendor = Device.get_info<device::vendor>();

      return Device.is_gpu() && DeviceName.find("HD Graphics NEO") ? 1 : -1;
    }
  };

  NEOGPUDeviceSelector Selector;
  try {
    cl::sycl::queue Queue(Selector);
    cl::sycl::device Device(Selector);
  } catch (cl::sycl::invalid_parameter_error &E) {
    std::cout << E.what() << std::endl;
  }
}

```

---
---
---
# C++ standard
- Minimally support C++ standard is c++11 on Linux and c++14 on Windows.

# Known Issues or Limitations

- SYCL device compiler fails if the same kernel was used in different
  translation units.
- SYCL host device is not fully supported.
- SYCL works only with OpenCL implementations supporting out-of-order queues.

# Find More

SYCL 1.2.1 specification: [www.khronos.org/registry/SYCL/specs/sycl-1.2.1.pdf](https://www.khronos.org/registry/SYCL/specs/sycl-1.2.1.pdf)
