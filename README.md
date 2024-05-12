## Compile FastAPI (ONNX GPURuntime) to executable binary (EXE)

### Background

I am working on a pose estimation desktop application - [juxt.space](https://github.com/ziqinyeow/juxt.space). 

My tech stack are [React](https://react.dev/)/[Vite](https://vitejs.dev/) as frontend, [FastAPI](https://fastapi.tiangolo.com/) as backend, [Tauri](https://tauri.app/) with GitHub Actions to auto-publish Windows/Mac/Ubuntu executables on release created.

<img width="639" alt="CleanShot 2024-05-11 at 22 28 39@2x" src="https://github.com/ziqinyeow/fastapi-onnx-gpu-exe/assets/74515561/5e0ec5aa-ac42-43e6-bfc7-445b10ba8730">

\
I was struggling to integrate the FastAPI (CPU works fine, but GPU no) server as a [Tauri Embedded Binary](https://tauri.app/v1/guides/building/sidecar/), because there's no existing solution to port onnx gpu runtime to exe.

I thought it might be useful to share some tips/knowledge on how to achieve this because, during the time, there's no solution for this.

### Target Audience

Windows users with GPU only for now (haven't tested for Ubuntu-GPU), most likely the same steps.

### Use Case

1. Run an ONNX GPU inference server without installing any Python dependencies.
2. Serve as a Tauri Sidecar/Embedded EXE to empower JavaScript/Rust applications.
3. Prevent end users from installing the Nvidia Cuda Toolkit (works well with GPU).

### Python + Pyinstaller

We are using [Pyinstaller](https://pyinstaller.org/en/stable/) to compile python script to executable binary. Make sure you have your FastAPI server.py ready. You can use this [server](https://github.com/ziqinyeow/juxtapose/blob/main/examples/fastapi-pyinstaller/) as a headstart.

```bash
pip install pyinstaller onnxruntime-gpu
```

<br />


### Step by Step to achieve this

The below command won't work in Windows. For some reason, pyinstaller in Windows can't trace some of the imports. You will need to manually build and run the exe, and debug which module is not found. 

<br />


```bash
pyinstaller -c -F --clean --name sidecar-x86_64-pc-windows-msvc --specpath dist --distpath dist server.py
```

<br />

In this case, I found out that when I run the above command, it produce an .exe file, but when I run the .exe file, it excited due to `cv2: Module Not Found`. 
Then I manually added in `--hidden-import=cv2`, and it worked.

<br />

```bash
pyinstaller -c -F --clean --hidden-import=cv2 --name sidecar-x86_64-pc-windows-msvc --specpath dist --distpath dist server.py
```

<br />


### GPU

After all, you will notice that when you run your server.exe and set the onnx device to `cuda`. You will face tons of errors like:

`CUDA_PATH is set but CUDA wasn't able to be loaded` - [Github Issue](https://github.com/microsoft/onnxruntime/issues/13576)

After reading all the Github Issues/threads, you will realize that there's no existing solution out there.

Maybe I need to ask all of my end users to install Nvidia CUDNN toolkits? Hmm, that is not ideal.

### After a week

I stumbled across some random China Blog Posts and realized that I need to include some `onnxruntime_*.dll` into that executable. Bam, problem solved.

Unzip this: https://github.com/microsoft/onnxruntime/releases/download/v1.17.3/onnxruntime-win-x64-gpu-cuda12-1.17.3.zip,

Add these three lines to your `pyinstaller` command

```bash
--add-binary="./onnxruntime_providers_cuda.dll;./onnxruntime/capi/" \
--add-binary="./onnxruntime_providers_tensorrt.dll;./onnxruntime/capi/" \
--add-binary="./onnxruntime_providers_shared.dll;./onnxruntime/capi/"
```

OR you can find these three files in your `envs\{env_name}\Lib\site-packages\onnxruntime\capi\*.dll`.

Your FastAPI exe now supports GPU runtime without extra config.


