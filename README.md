# Compile FastAPI (ONNX GPURuntime) to executable binary

## Background

I am working on a pose estimation desktop application - [juxt.space](https://github.com/ziqinyeow/juxt.space). 

My tech stack includes [React](https://react.dev/)/[Vite](https://vitejs.dev/) as frontend, [FastAPI](https://fastapi.tiangolo.com/) as backend, [Tauri](https://tauri.app/) with GitHub Actions to auto-publish Windows/Mac/Ubuntu executables on release created.

<img width="639" alt="CleanShot 2024-05-11 at 22 28 39@2x" src="https://github.com/ziqinyeow/fastapi-onnx-gpu-exe/assets/74515561/5e0ec5aa-ac42-43e6-bfc7-445b10ba8730">

\
I was struggling to integrate the FastAPI (CPU works fine, but GPU no) server as a [Tauri Embedded Binary](https://tauri.app/v1/guides/building/sidecar/), because there's no existing solution to port onnx gpu runtime to exe.

I thought it might be useful to share some tips/knowledge on how to achieve this because, during the time, there's no solution for this.

## Target Audience

Windows users with GPU only for now (haven't tested for Ubuntu-GPU), most likely the same steps.

## Use Case

1. Run an ONNX GPU inference server without installing any Python dependencies.
2. Serve as a Tauri Sidecar/Embedded EXE to empower JavaScript/Rust applications.

## Stack to achieve this

Make sure you have your FastAPI server ready. You can use this [server](https://github.com/ziqinyeow/juxtapose/blob/main/examples/fastapi-pyinstaller/) as a headstart.

```bash
pip install pyinstaller
```
