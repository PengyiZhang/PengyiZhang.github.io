  Loading a TorchScript Model in C++ — PyTorch Tutorials 1.4.0 documentation                      


As its name suggests, the primary interface to PyTorch is the Python programming language. While Python is a suitable and preferred language for many scenarios requiring dynamism and ease of iteration, there are equally many situations where precisely these properties of Python are unfavorable. One environment in which the latter often applies is _production_ – the land of low latencies and strict deployment requirements. For production scenarios, C++ is very often the language of choice, even if only to bind it into another language like Java, Rust or Go. The following paragraphs will outline the path PyTorch provides to go from an existing Python model to a serialized representation that can be _loaded_ and _executed_ purely from C++, with no dependency on Python.

Step 1: Converting Your PyTorch Model to Torch Script[¶](#step-1-converting-your-pytorch-model-to-torch-script "Permalink to this headline")
--------------------------------------------------------------------------------------------------------------------------------------------

A PyTorch model’s journey from Python to C++ is enabled by [Torch Script](https://pytorch.org/docs/master/jit.html), a representation of a PyTorch model that can be understood, compiled and serialized by the Torch Script compiler. If you are starting out from an existing PyTorch model written in the vanilla “eager” API, you must first convert your model to Torch Script. In the most common cases, discussed below, this requires only little effort. If you already have a Torch Script module, you can skip to the next section of this tutorial.

There exist two ways of converting a PyTorch model to Torch Script. The first is known as _tracing_, a mechanism in which the structure of the model is captured by evaluating it once using example inputs, and recording the flow of those inputs through the model. This is suitable for models that make limited use of control flow. The second approach is to add explicit annotations to your model that inform the Torch Script compiler that it may directly parse and compile your model code, subject to the constraints imposed by the Torch Script language.

Tip

You can find the complete documentation for both of these methods, as well as further guidance on which to use, in the official [Torch Script reference](https://pytorch.org/docs/master/jit.html).

### Converting to Torch Script via Tracing[¶](#converting-to-torch-script-via-tracing "Permalink to this headline")

To convert a PyTorch model to Torch Script via tracing, you must pass an instance of your model along with an example input to the `torch.jit.trace` function. This will produce a `torch.jit.ScriptModule` object with the trace of your model evaluation embedded in the module’s `forward` method:

import torch
import torchvision

\# An instance of your model.
model = torchvision.models.resnet18()

\# An example input you would normally provide to your model's forward() method.
example = torch.rand(1, 3, 224, 224)

\# Use torch.jit.trace to generate a torch.jit.ScriptModule via tracing.
traced\_script\_module = torch.jit.trace(model, example)

The traced `ScriptModule` can now be evaluated identically to a regular PyTorch module:

In\[1\]: output = traced\_script\_module(torch.ones(1, 3, 224, 224))
In\[2\]: output\[0, :5\]
Out\[2\]: tensor(\[-0.2698, -0.0381,  0.4023, -0.3010, -0.0448\], grad_fn=<SliceBackward>)

### Converting to Torch Script via Annotation[¶](#converting-to-torch-script-via-annotation "Permalink to this headline")

Under certain circumstances, such as if your model employs particular forms of control flow, you may want to write your model in Torch Script directly and annotate your model accordingly. For example, say you have the following vanilla Pytorch model:

import torch

class MyModule(torch.nn.Module):
    def \_\_init\_\_(self, N, M):
        super(MyModule, self).\_\_init\_\_()
        self.weight = torch.nn.Parameter(torch.rand(N, M))

    def forward(self, input):
        if input.sum() > 0:
          output = self.weight.mv(input)
        else:
          output = self.weight + input
        return output

Because the `forward` method of this module uses control flow that is dependent on the input, it is not suitable for tracing. Instead, we can convert it to a `ScriptModule`. In order to convert the module to the `ScriptModule`, one needs to compile the module with `torch.jit.script` as follows:

class MyModule(torch.nn.Module):
    def \_\_init\_\_(self, N, M):
        super(MyModule, self).\_\_init\_\_()
        self.weight = torch.nn.Parameter(torch.rand(N, M))

    def forward(self, input):
        if input.sum() > 0:
          output = self.weight.mv(input)
        else:
          output = self.weight + input
        return output

my_module = MyModule(10,20)
sm = torch.jit.script(my_module)

If you need to exclude some methods in your `nn.Module` because they use Python features that TorchScript doesn’t support yet, you could annotate those with `@torch.jit.ignore`

`my_module` is an instance of `ScriptModule` that is ready for serialization.

Step 2: Serializing Your Script Module to a File[¶](#step-2-serializing-your-script-module-to-a-file "Permalink to this headline")
----------------------------------------------------------------------------------------------------------------------------------

Once you have a `ScriptModule` in your hands, either from tracing or annotating a PyTorch model, you are ready to serialize it to a file. Later on, you’ll be able to load the module from this file in C++ and execute it without any dependency on Python. Say we want to serialize the `ResNet18` model shown earlier in the tracing example. To perform this serialization, simply call [save](https://pytorch.org/docs/master/jit.html#torch.jit.ScriptModule.save) on the module and pass it a filename:

traced\_script\_module.save("traced\_resnet\_model.pt")

This will produce a `traced_resnet_model.pt` file in your working directory. If you also would like to serialize `my_module`, call `my_module.save("my_module_model.pt")` We have now officially left the realm of Python and are ready to cross over to the sphere of C++.

Step 3: Loading Your Script Module in C++[¶](#step-3-loading-your-script-module-in-c "Permalink to this headline")
------------------------------------------------------------------------------------------------------------------

To load your serialized PyTorch model in C++, your application must depend on the PyTorch C++ API – also known as _LibTorch_. The LibTorch distribution encompasses a collection of shared libraries, header files and CMake build configuration files. While CMake is not a requirement for depending on LibTorch, it is the recommended approach and will be well supported into the future. For this tutorial, we will be building a minimal C++ application using CMake and LibTorch that simply loads and executes a serialized PyTorch model.

### A Minimal C++ Application[¶](#a-minimal-c-application "Permalink to this headline")

Let’s begin by discussing the code to load a module. The following will already do:

#include <torch/script.h> // One-stop header.

#include <iostream>
#include <memory>

int main(int argc, const char* argv\[\]) {
  if (argc != 2) {
    std::cerr << "usage: example-app <path-to-exported-script-module>\\n";
    return -1;
  }

  torch::jit::script::Module module;
  try {
    // Deserialize the ScriptModule from a file using torch::jit::load().
    module = torch::jit::load(argv\[1\]);
  }
  catch (const c10::Error& e) {
    std::cerr << "error loading the model\\n";
    return -1;
  }

  std::cout << "ok\\n";
}

The `<torch/script.h>` header encompasses all relevant includes from the LibTorch library necessary to run the example. Our application accepts the file path to a serialized PyTorch `ScriptModule` as its only command line argument and then proceeds to deserialize the module using the `torch::jit::load()` function, which takes this file path as input. In return we receive a `torch::jit::script::Module` object. We will examine how to execute it in a moment.

### Depending on LibTorch and Building the Application[¶](#depending-on-libtorch-and-building-the-application "Permalink to this headline")

Assume we stored the above code into a file called `example-app.cpp`. A minimal `CMakeLists.txt` to build it could look as simple as:

cmake\_minimum\_required(VERSION 3.0 FATAL_ERROR)
project(custom_ops)

find_package(Torch REQUIRED)

add_executable(example-app example-app.cpp)
target\_link\_libraries(example-app "${TORCH_LIBRARIES}")
set_property(TARGET example-app PROPERTY CXX_STANDARD 14)

The last thing we need to build the example application is the LibTorch distribution. You can always grab the latest stable release from the [download page](https://pytorch.org/) on the PyTorch website. If you download and unzip the latest archive, you should receive a folder with the following directory structure:

libtorch/
  bin/
  include/
  lib/
  share/

*   The `lib/` folder contains the shared libraries you must link against,
*   The `include/` folder contains header files your program will need to include,
*   The `share/` folder contains the necessary CMake configuration to enable the simple `find_package(Torch)` command above.

Tip

On Windows, debug and release builds are not ABI-compatible. If you plan to build your project in debug mode, please try the debug version of LibTorch. Also, make sure you specify the correct configuration in the `cmake --build .` line below.

The last step is building the application. For this, assume our example directory is laid out like this:

example-app/
  CMakeLists.txt
  example-app.cpp

We can now run the following commands to build the application from within the `example-app/` folder:

mkdir build
cd build
cmake -DCMAKE\_PREFIX\_PATH=/path/to/libtorch ..
cmake --build . --config Release

where `/path/to/libtorch` should be the full path to the unzipped LibTorch distribution. If all goes well, it will look something like this:

root@4b5a67132e81:/example-app# mkdir build
root@4b5a67132e81:/example-app# cd build
root@4b5a67132e81:/example-app/build# cmake -DCMAKE\_PREFIX\_PATH=/path/to/libtorch ..
\-\- The C compiler identification is GNU 5.4.0
\-\- The CXX compiler identification is GNU 5.4.0
\-\- Check for working C compiler: /usr/bin/cc
\-\- Check for working C compiler: /usr/bin/cc -- works
\-\- Detecting C compiler ABI info
\-\- Detecting C compiler ABI info - done
\-\- Detecting C compile features
\-\- Detecting C compile features - done
\-\- Check for working CXX compiler: /usr/bin/c++
\-\- Check for working CXX compiler: /usr/bin/c++ -- works
\-\- Detecting CXX compiler ABI info
\-\- Detecting CXX compiler ABI info - done
\-\- Detecting CXX compile features
\-\- Detecting CXX compile features - done
\-\- Looking for pthread.h
\-\- Looking for pthread.h - found
\-\- Looking for pthread_create
\-\- Looking for pthread_create - not found
\-\- Looking for pthread_create in pthreads
\-\- Looking for pthread_create in pthreads - not found
\-\- Looking for pthread_create in pthread
\-\- Looking for pthread_create in pthread - found
\-\- Found Threads: TRUE
\-\- Configuring done
\-\- Generating done
\-\- Build files have been written to: /example-app/build
root@4b5a67132e81:/example-app/build# make
Scanning dependencies of target example-app
\[ 50%\] Building CXX object CMakeFiles/example-app.dir/example-app.cpp.o
\[100%\] Linking CXX executable example-app
\[100%\] Built target example-app

If we supply the path to the traced `ResNet18` model `traced_resnet_model.pt` we created earlier to the resulting `example-app` binary, we should be rewarded with a friendly “ok”. Please note, if try to run this example with `my_module_model.pt` you will get an error saying that your input is of an incompatible shape. `my_module_model.pt` expects 1D instead of 4D.

root@4b5a67132e81:/example-app/build# ./example-app <path\_to\_model>/traced\_resnet\_model.pt
ok

Step 4: Executing the Script Module in C++[¶](#step-4-executing-the-script-module-in-c "Permalink to this headline")
--------------------------------------------------------------------------------------------------------------------

Having successfully loaded our serialized `ResNet18` in C++, we are now just a couple lines of code away from executing it! Let’s add those lines to our C++ application’s `main()` function:

// Create a vector of inputs.
std::vector<torch::jit::IValue> inputs;
inputs.push_back(torch::ones({1, 3, 224, 224}));

// Execute the model and turn its output into a tensor.
at::Tensor output = module.forward(inputs).toTensor();
std::cout << output.slice(/\*dim=\*/1, /\*start=\*/0, /\*end=\*/5) << '\\n';

The first two lines set up the inputs to our model. We create a vector of `torch::jit::IValue` (a type-erased value type `script::Module` methods accept and return) and add a single input. To create the input tensor, we use `torch::ones()`, the equivalent to `torch.ones` in the C++ API. We then run the `script::Module`’s `forward` method, passing it the input vector we created. In return we get a new `IValue`, which we convert to a tensor by calling `toTensor()`.

Tip

To learn more about functions like `torch::ones` and the PyTorch C++ API in general, refer to its documentation at [https://pytorch.org/cppdocs](https://pytorch.org/cppdocs). The PyTorch C++ API provides near feature parity with the Python API, allowing you to further manipulate and process tensors just like in Python.

In the last line, we print the first five entries of the output. Since we supplied the same input to our model in Python earlier in this tutorial, we should ideally see the same output. Let’s try it out by re-compiling our application and running it with the same serialized model:

root@4b5a67132e81:/example-app/build# make
Scanning dependencies of target example-app
\[ 50%\] Building CXX object CMakeFiles/example-app.dir/example-app.cpp.o
\[100%\] Linking CXX executable example-app
\[100%\] Built target example-app
root@4b5a67132e81:/example-app/build# ./example-app traced\_resnet\_model.pt
-0.2698 -0.0381  0.4023 -0.3010 -0.0448
\[ Variable\[CPUFloatType\]{1,5} \]

For reference, the output in Python previously was:

tensor(\[-0.2698, -0.0381,  0.4023, -0.3010, -0.0448\], grad_fn=<SliceBackward>)

Looks like a good match!

Tip

To move your model to GPU memory, you can write `model.to(at::kCUDA);`. Make sure the inputs to a model are also living in CUDA memory by calling `tensor.to(at::kCUDA)`, which will return a new tensor in CUDA memory.

Step 5: Getting Help and Exploring the API[¶](#step-5-getting-help-and-exploring-the-api "Permalink to this headline")
----------------------------------------------------------------------------------------------------------------------

This tutorial has hopefully equipped you with a general understanding of a PyTorch model’s path from Python to C++. With the concepts described in this tutorial, you should be able to go from a vanilla, “eager” PyTorch model, to a compiled `ScriptModule` in Python, to a serialized file on disk and – to close the loop – to an executable `script::Module` in C++.

Of course, there are many concepts we did not cover. For example, you may find yourself wanting to extend your `ScriptModule` with a custom operator implemented in C++ or CUDA, and executing this custom operator inside your `ScriptModule` loaded in your pure C++ production environment. The good news is: this is possible, and well supported! For now, you can explore [this](https://github.com/pytorch/pytorch/tree/master/test/custom_operator) folder for examples, and we will follow up with a tutorial shortly. In the time being, the following links may be generally helpful:

*   The Torch Script reference: [https://pytorch.org/docs/master/jit.html](https://pytorch.org/docs/master/jit.html)
*   The PyTorch C++ API documentation: [https://pytorch.org/cppdocs/](https://pytorch.org/cppdocs/)
*   The PyTorch Python API documentation: [https://pytorch.org/docs/](https://pytorch.org/docs/)

