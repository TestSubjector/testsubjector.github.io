---
layout: post
title: "Setting Up TensorFlow On Julia"
date: 2017-11-12
---
----------------
*Note* - The instructions are chiefly for Linux (Ubuntu) users. Apologies to others. Also, beware that the steps for a GPU enabled version of TensorFlow is fraught with risks, thanks to the breakage prone shenanigans of Nvidia drivers on Ubuntu/Linux.  
  
First, a few definitions.  
  
**What is [TensorFlow](https://www.tensorflow.org)?**  
TensorFlow is an open-source software library for dataflow programming across a range of tasks. It is a 
symbolic math library, and is also used for machine learning applications such as neural networks.  
  
**What is [TensorFlow,jl](https://github.com/malmaud/TensorFlow.jl)?**  
A wrapper around TensorFlow in the Julia language. Also, [some advantages of this](https://github.com/malmaud/TensorFlow.jl/blob/master/docs/src/why_julia.md) over Python.  
  
  
Now, let's get everything set up.  
  
+ Firstly, you need to [download Julia](https://julialang.org/downloads/). For more complex ways of installation, [see here](https://github.com/JuliaLang/julia#source-download-and-compilation). *Note* - Download version 0.6 as it is the latest version of Julia supported for TensorFlow.jl; valid as of the date of this blog post being uploaded.  
  

**For CPU version of TensorFlow**  
+ Now, provided you only need the *CPU version* (don't have or want to use an Nvidia graphic card), first access the Julia REPL. For those who downloaded the binaries, they can find the julia executable in bin folder of the downloaded files. Now, do the following in the REPL -  
`julia> Pkg.add("TensorFlow")`  # In the Julia REPL  
`julia> Pkg.build("TensorFlow")`  # In the Julia REPL  
  
That should do it. Try typing `using TensorFlow` in the REPL to check if the package is running perfectly. Refer to the link ahead in case you see [issues from the ccall or python interop](https://github.com/malmaud/TensorFlow.jl#troubleshooting).  
  

**For GPU-enabled version of TensorFlow**  
  
**The Nvidia Driver**  
  
Right, that was the simple route for CPU usage. You can optimise CPU usage by [building TensorFlow from source](https://malmaud.github.io/TensorFlow.jl/latest/build_from_source.html). That's a bit outside the scope of a simple setup though.    
Now if you want to use the shiny Nvidia graphics card i.e use *TensorFlow with GPU enabled*, there's going to be a bit of pain. In the range of possible scenarios, you could faces problems ranging from lower screen resolution to a flickering screen to a complete black screen.  
  
The tricky thing is there's no one-shot cure to this problem. Here's the standard method -  
`sudo add-apt-repository ppa:graphics-drivers/ppa`  
`sudo apt-get update`  
`sudo apt-get install nvidia-current`  
`sudo reboot`  
  
You'll be asked to enter a password to disable the secure boot after login. Be careful to not forget the password, in case of problems it can come handy.  
If you do face problems and are not sure what to do, don't panic. Try to access the [TTY](https://askubuntu.com/questions/66195/what-is-a-tty-and-how-do-i-access-a-tty) and type in the following command to get back to how things were before -  
`sudo apt-get purge nvidia-current`  
`sudo reboot`  
  
As I said, different people with different setups + hardware + driver version will face different problems.  
The thing that worked for me personally (after hours of successive install and purge cycles) was to install the drivers not from the command line but from the GUI of the Additional Drivers application.  
I simply selected the version(381) I wanted (select the circle on the left), which it automatically installed. Then, Reboot -> disable secure boot -> Voila!  
Only thing I can say is, good luck with this step.  
  

**CUDA  and cuDNN**  
+ I used [this complete guide by Nvidia itself](https://www.nvidia.com/en-us/data-center/gpu-accelerated-applications/tensorflow/) to download these two components. Just one thing you have to make sure is that the CUDA version you download should be 8.x and the cuDNN version should be v5.x For other versions, [this may help](https://github.com/malmaud/TensorFlow.jl#optional-building-the-tensorflow-library).  
  
  
**Final step**  
+ If everything is holding together till now, then all you should need to do is -  
`julia> Pkg.add("TensorFlow")`  # In the Julia REPL  
`julia> ENV["TF_USE_GPU"] = "1"`  # In the Julia REPL  
`julia> Pkg.build("TensorFlow")`  # In the Julia REPL  
If typing `using TensorFlow` and then running [this code](https://github.com/malmaud/TensorFlow.jl#basic-usage) doesn't give an error, then congratulations! You've made it through the setup.  
The actual coding now awaits.  
