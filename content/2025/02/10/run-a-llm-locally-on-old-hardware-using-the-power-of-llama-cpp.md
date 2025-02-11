---
title: Run a LLM Locally on Old Hardware Using The Power of llama.cpp
date: 2025-02-10
draft: false 
tags:
  - llm
  - build
  - macOS
  - brew
  - cmake
  - vulkan
  - amd
  - intel 
---

> I have a Mac with Intel silicon. I also have an eGPU with an AMD 6900XT (...allright!). BUT I COULDN'T HARNESS THAT POWER AND RUN A LLM LOCALLY WITH OLLAMA!!! If you have a Mac with Intel silicon, then you know that the CPU and integrated GPU are insufficient for running a LLM locally. I don't want to buy new hardware to play around with LLMs locally. Then I found [llama.cpp](https://github.com/ggerganov/llama.cpp). This is an amazing repo that helps democratize the use of LLMs on local machines. However, the install instructions on the [llama.cpp](https://github.com/ggerganov/llama.cpp) repo did not cover the issues I came across, which is why I was motivated to write up a post to help others out. If you have older hardware that isn't supported by the current tools to run a LLM locally (specifically a Mac with Intel silicon and an AMD eGPU), then this post is for you! As a side note, if your hardware isn't a 1:1 match for what I have, but you realize that running the Vulkan SDK backend for [llama.cpp](https://github.com/ggerganov/llama.cpp) is the right fit for you, then this post may be useful for setting up the Vulkan backend.

## 0. Dependencies and Setup

* If you have a Mac with Intel silicon, then you need to use the `Vulkan` backend when setting up [llama.cpp](https://github.com/ggerganov/llama.cpp) because [llama.cpp](https://github.com/ggerganov/llama.cpp) only supports the `Metal` api on Macs for Apple silicon and AMD only provides `HIP` support for Linux.
* Make sure you haven't installed `MoltenVK` or other `Vulkan` SDK components piecemeal via a package manager like `brew` because that can interfere with the `Vulkan` SDK install.
* Download and install the [Vulkan SDK](https://vulkan.lunarg.com/sdk/home).
* Install `cmake` and `libomp` with your favorite package manager.
* Make sure you verify the `sha256` hash to ensure the file you downloaded is correct.

```sh
# verify the sha256 hash on your local machine, I'll do it on my Mac with the below command
openssl dgst -sha256 ./path/to/download/vulkansdk-macos-1.4.304.0.zip

# verify the output visually, on a repl, etc

# you will also need to install cmake and libomp
brew install cmake libomp

brew doctor --verbose
# if there is no output, then proceed
# if there are files listed here, copy them and save them in a separate txt file

# get the `llama.cpp` repo from github
git clone https://github.com/ggerganov/llama.cpp.git

# or with the github cli get the `llama.cpp` repo from github
# gh repo clone ggerganov/llama.cpp

# ensure you are in the llama.cpp directory for the next steps
cd llama.cpp
```

## 1. Build `llama.cpp` Using `cmake`

* When I tried to build `llama.cpp` with `cmake` for the first time following the [instructions on the llama.cpp repo for Vulkan](https://github.com/ggerganov/llama.cpp/blob/master/docs/build.md#vulkan), I got the following error after running `cmake -B build -DGGML_VULKAN=ON`:

```sh
-- Could NOT find OpenMP_C (missing: OpenMP_C_FLAGS OpenMP_C_LIB_NAMES) 
-- Could NOT find OpenMP_CXX (missing: OpenMP_CXX_FLAGS OpenMP_CXX_LIB_NAMES) 
-- Could NOT find OpenMP (missing: OpenMP_C_FOUND OpenMP_CXX_FOUND) 
CMake Warning at ggml/src/ggml-cpu/CMakeLists.txt:53 (message):
  OpenMP not found
```

* On your Mac, Xcode Command Line Tools or `xcode-select` should provide access to [OpenMP](https://en.wikipedia.org/wiki/OpenMP) via `Clang`.
* My version of `cmake` could not see the `OpenMP` api included with `Clang`, so to make my life easier I just installed `OpenMP` via `brew install libomp`.
* After this, I needed to link `libomp` when running `cmake -B build -DGGML_VULKAN=ON` from the [instructions on the llama.cpp repo for Vulkan](https://github.com/ggerganov/llama.cpp/blob/master/docs/build.md#vulkan). Since, I installed `libomp` with `brew`, the pathway to `libomp` reflects that. Also, I explicitly ensured the `Metal` api was off.

```sh
cmake -B build -DGGML_METAL=OFF -DGGML_VULKAN=ON \
-DOpenMP_C_FLAGS=-fopenmp=lomp \
-DOpenMP_CXX_FLAGS=-fopenmp=lomp \
-DOpenMP_C_LIB_NAMES="libomp" \
-DOpenMP_CXX_LIB_NAMES="libomp" \
-DOpenMP_libomp_LIBRARY="$(brew --prefix)/opt/libomp/lib/libomp.dylib" \
-DOpenMP_CXX_FLAGS="-Xpreprocessor -fopenmp $(brew --prefix)/opt/libomp/lib/libomp.dylib -I$(brew --prefix)/opt/libomp/include" \
-DOpenMP_CXX_LIB_NAMES="libomp" \
-DOpenMP_C_FLAGS="-Xpreprocessor -fopenmp $(brew --prefix)/opt/libomp/lib/libomp.dylib -I$(brew --prefix)/opt/libomp/include"
```

* I didn't run into any other errors after running the previous command, but make sure to read through the output to ensure everything is fine on your system.
* To complete the build process, run `cmake --build build --config Release`.
* You will see many objects being built so that `llama.cpp` can run, and the following warnings as of version `b4686` of `llama.cpp`:

```sh
ggml_vulkan: Generating and compiling shaders to SPIR-V
[  6%] Building CXX object ggml/src/ggml-vulkan/CMakeFiles/ggml-vulkan.dir/ggml-vulkan.cpp.o
/Users/ankit/Playground/llama.cpp/ggml/src/ggml-vulkan/ggml-vulkan.cpp:1382:2: warning: extra ';' outside of a function is incompatible with C++98 [-Wc++98-compat-extra-semi]
 1382 | };
      |  ^
/Users/ankit/Playground/llama.cpp/ggml/src/ggml-vulkan/ggml-vulkan.cpp:7048:16: warning: 'return' will never be executed [-Wunreachable-code-return]
 7048 |         return false;
      |                ^~~~~
/Users/ankit/Playground/llama.cpp/ggml/src/ggml-vulkan/ggml-vulkan.cpp:8208:15: warning: 'break' will never be executed [-Wunreachable-code-break]
 8208 |             } break;
      |               ^~~~~
/Users/ankit/Playground/llama.cpp/ggml/src/ggml-vulkan/ggml-vulkan.cpp:8167:15: warning: 'break' will never be executed [-Wunreachable-code-break]
 8167 |             } break;
      |               ^~~~~
/Users/ankit/Playground/llama.cpp/ggml/src/ggml-vulkan/ggml-vulkan.cpp:8088:15: warning: 'break' will never be executed [-Wunreachable-code-break]
 8088 |             } break;
      |               ^~~~~
/Users/ankit/Playground/llama.cpp/ggml/src/ggml-vulkan/ggml-vulkan.cpp:8035:13: warning: 'break' will never be executed [-Wunreachable-code-break]
 8035 |             break;
      |             ^~~~~
6 warnings generated.
[  6%] Building CXX object ggml/src/ggml-vulkan/CMakeFiles/ggml-vulkan.dir/ggml-vulkan-shaders.cpp.o
[  7%] Linking CXX shared library ../../../bin/libggml-vulkan.dylib
[  7%] Built target ggml-vulkan
[  8%] Building C object ggml/src/CMakeFiles/ggml-cpu.dir/ggml-cpu/ggml-cpu.c.o
cc: warning: /usr/local/opt/libomp/lib/libomp.dylib: 'linker' input unused [-Wunused-command-line-argument]
In file included from /Users/ankit/Playground/llama.cpp/ggml/src/ggml-cpu/ggml-cpu.c:40:
/usr/local/opt/libomp/include/omp.h:54:9: warning: ISO C restricts enumerator values to range of 'int' (2147483648 is too large) [-Wpedantic]
   54 |         omp_sched_monotonic = 0x80000000
      |         ^                     ~~~~~~~~~~
/usr/local/opt/libomp/include/omp.h:411:7: warning: ISO C restricts enumerator values to range of 'int' (18446744073709551615 is too large) [-Wpedantic]
  411 |       KMP_ALLOCATOR_MAX_HANDLE = UINTPTR_MAX
      |       ^                          ~~~~~~~~~~~
/usr/local/opt/libomp/include/omp.h:427:7: warning: ISO C restricts enumerator values to range of 'int' (18446744073709551615 is too large) [-Wpedantic]
  427 |       KMP_MEMSPACE_MAX_HANDLE = UINTPTR_MAX
      |       ^                         ~~~~~~~~~~~
/usr/local/opt/libomp/include/omp.h:471:39: warning: ISO C restricts enumerator values to range of 'int' (18446744073709551615 is too large) [-Wpedantic]
  471 |     typedef enum omp_event_handle_t { KMP_EVENT_MAX_HANDLE = UINTPTR_MAX } omp_event_handle_t;
      |                                       ^                      ~~~~~~~~~~~
4 warnings generated.
[  8%] Building CXX object ggml/src/CMakeFiles/ggml-cpu.dir/ggml-cpu/ggml-cpu.cpp.o
c++: warning: /usr/local/opt/libomp/lib/libomp.dylib: 'linker' input unused [-Wunused-command-line-argument]
[  9%] Building CXX object ggml/src/CMakeFiles/ggml-cpu.dir/ggml-cpu/ggml-cpu-aarch64.cpp.o
c++: warning: /usr/local/opt/libomp/lib/libomp.dylib: 'linker' input unused [-Wunused-command-line-argument]
[  9%] Building CXX object ggml/src/CMakeFiles/ggml-cpu.dir/ggml-cpu/ggml-cpu-hbm.cpp.o
c++: warning: /usr/local/opt/libomp/lib/libomp.dylib: 'linker' input unused [-Wunused-command-line-argument]
[  9%] Building C object ggml/src/CMakeFiles/ggml-cpu.dir/ggml-cpu/ggml-cpu-quants.c.o
cc: warning: /usr/local/opt/libomp/lib/libomp.dylib: 'linker' input unused [-Wunused-command-line-argument]
[ 10%] Building CXX object ggml/src/CMakeFiles/ggml-cpu.dir/ggml-cpu/ggml-cpu-traits.cpp.o
c++: warning: /usr/local/opt/libomp/lib/libomp.dylib: 'linker' input unused [-Wunused-command-line-argument]
[ 10%] Building CXX object ggml/src/CMakeFiles/ggml-cpu.dir/ggml-cpu/amx/amx.cpp.o
c++: warning: /usr/local/opt/libomp/lib/libomp.dylib: 'linker' input unused [-Wunused-command-line-argument]
[ 11%] Building CXX object ggml/src/CMakeFiles/ggml-cpu.dir/ggml-cpu/amx/mmq.cpp.o
c++: warning: /usr/local/opt/libomp/lib/libomp.dylib: 'linker' input unused [-Wunused-command-line-argument]
[ 11%] Building CXX object ggml/src/CMakeFiles/ggml-cpu.dir/ggml-cpu/llamafile/sgemm.cpp.o
c++: warning: /usr/local/opt/libomp/lib/libomp.dylib: 'linker' input unused [-Wunused-command-line-argument]
[ 12%] Linking CXX shared library ../../bin/libggml-cpu.dylib
[ 12%] Built target ggml-cpu
```

* I have not run into any issues from these warnings as of yet. But, I will update this post in the future if I do.

## 2. Getting Models for `llama.cpp`

* You will need to download models to run with `llama.cpp`.
* [Hugging Face](https://huggingface.co/) is an excellent source for models, but make sure you get quantized models. Quantized models have been changed to work on hardware that does not have enough `RAM` to run the model as it was intended. The only change in the model is how large of an input the model will work with by [reducing the bits](https://huggingface.co/docs/optimum/en/concept_guides/quantization).
* I will create a future post regarding quantizing models, but for now we will use a pre-quantized model for the purposes of testing our build.
* `cd ../` or go one level up outside of the `llama.cpp` directory and `mkdir llm-models`. We will store all of our models outside of the `llama.cpp` repo.
* Go to the newly created directory `cd llm-models`.
* I downloaded an `8 bit` `Meta Llama 3.1` model from [ggml's hugging face](https://huggingface.co/ggml-org), which was quantized using a `Q4_0` quantization method.
  * [Meta-Llama-3.1-8B-Instruct-Q4_0-GGUF](https://huggingface.co/ggml-org/Meta-Llama-3.1-8B-Instruct-Q4_0-GGUF/resolve/main/meta-llama-3.1-8b-instruct-q4_0.gguf?download=true)
* Ensure you download the model into the `llm-models/` directory.
* Once the download is complete, we are ready to run the model locally.

## 3. Running `llama.cpp`

```sh
cd llama.cpp

# start interactive mode
./build/bin/llama-cli -m ../llm-models/meta-llama-3.1-8b-instruct-q4_0.gguf
```

* If everything went well, `llama.cpp` will see the AMD GPU:

```sh
ggml_vulkan: Found 1 Vulkan devices:
ggml_vulkan: 0 = AMD Radeon RX 6900 XT (MoltenVK) | uma: 0 | fp16: 1 | warp size: 64 | shared memory: 65536 | matrix cores: none
```

* Test out interactive mode and have fun running an `LLM` locally!

## 4. Last Minute Cleanup

* On my Mac, the `Vulkan` SDK created a bunch of `dylib`, `static lib`, `pc`, and `header` files in `/usr/local/lib`.
* When you run `brew doctor --verbose`, `brew` will give you a bunch of warnings that it found unbrewed files.
* You can choose to ignore this warning. However, if it bothers you like it bothered me. You'll want to do something about it.
* `WARNING:` The next steps involve altering `brew` locally, and is a temporary fix. If you are not comfortable working with `bash` functions or altering `ruby` code, do not proceed.
* Go to `/usr/local/Homebrew/Library/Homebrew`.
* There you will find `diagnostic.rb`.
* Open this file and change the `allow_list` array in the following functions `def check_for_stray_dylibs`, `def check_for_stray_static_libs`, `def check_for_stray_pcs`, and `def check_for_stray_headers`, with the corresponding files. You can run `brew doctor --verbose` to get the list of files again.
* Make sure not to include the files that were output by `brew doctor --verbose` and saved to a separate file. If there were unbrewed files before installing the `Vulkan` SDK, then they must be addressed separately and are not part of the scope of this post.
* Once you have changed `diagnostic.rb`, save the file.
* When you `cd /usr/local/Homebrew` and run `git status`, you will see that the `diagnostic.rb` is changed. We cannot commit these changes.
* If you run `brew doctor --verbose` the files added to `/usr/local/lib` by the `Vulkan` SDK are no longer there.
* These changes are not permanent. To ensure I don't have to edit `diagnostic.rb` each time I upgrade `brew`. I wrote two `bash` functions and saved the list of `dylibs`, `static libs`, `pcs`, and `headers` to a `JSON` file.
* Add the following functions to your `.bashrc`, `.zshrc` or `/custom` for `oh-my-zsh`

```bash
function allow-stash()
{
  CURRDIRR=$(pwd)
  cd /usr/local/HomeBrew
  git stash
  cd $CURRDIRR
}

function allow-stash-apply()
{
  CURRDIRR=$(pwd)
  cd /usr/local/HomeBrew
  git stash apply
  cd $CURRDIRR
}
```

* `allow-stash` saves the changes made to the `diagnostic.rb` in `git stash`. We're simply saving the appends made to the `allow_list` in the previously listed functions.
* Then I run `brew update && brew upgrade`.
* Then I run `allow-stash-apply` to add my changes to `diagnostic.rb` again.
* You can create a third function to combine all these steps into a single command.

```bash
function update-brew()
{
  allow-stash && brew update && brew upgrade && allow-stash-apply
}
```

* `bash` will run all these commands in sequence from left to right, so the order matters.

## 5. References

* [llama.cpp](https://github.com/ggerganov/llama.cpp)
* [Hugging Face](https://huggingface.co/)
* [homebrew](https://brew.sh/)
* [bash docs](https://www.gnu.org/software/bash/manual/bash.html)
