---
title: How to Deploy a Fast One-Click Face Swapping Software on Hugging Face
date: 2023-07-04 15:07:06
tags: tools
---



## 1. Introduction to Hugging Face

[Hugging Face](https://huggingface.co/) is a platform that provides powerful tools and resources for the natural language processing (NLP) community. It offers various pre-trained models, APIs, and tools to simplify the development and deployment of NLP tasks for developers and researchers. The core mission of Hugging Face is to make NLP technology more accessible and shareable, promoting the advancement and democratization of artificial intelligence.

On Hugging Face, users can access a wide range of pre-trained models for tasks like text classification, question answering, language generation, and more. Additionally, Hugging Face allows users to create their own "Spaces" to store and share their models, code, and datasets. In this guide, we will focus on how to deploy a fast one-click face swapping software on Hugging Face.

## 2. Creating a Space and Uploading Code

To deploy a one-click face swapping software on Hugging Face, we first need to create a Hugging Face account and then create a new Space.

### 2.1 Create a Hugging Face Account

If you don't have a Hugging Face account yet, head to the [Hugging Face registration page](https://huggingface.co/join) and create a new account.

### 2.2 Login and Create a New Space

1. Login to [Hugging Face](https://huggingface.co/), click the "Login" button in the top right corner, and enter your account credentials to log in.

2. After logging in, you will see your username in the top right corner. Click on your username and select "Create a new space.". and select the Space SDK Gradio
for space hardware, we will be using CPU.

3. On the "Create a new space" page, provide a name and description for your Space. Choose a relevant name related to your one-click face swapping software and provide an appropriate description to let others know about your project.

4. Click "Create new space" to finish creating the Space.
![create-huggingface-space](/images/create-hugging-face-space.png)

### 2.3 Upload Code and Applications

To deploy our one-click face swapping software, we will be using the [roop](https://github.com/ezioruan/roop) repository available on GitHub. This software enables one-click face swapping for both images and videos by simply requiring users to upload a portrait image. For the purpose of this guide, we will focus on face swapping as an example. As we are using the free version of Hugging Face's space, it currently supports CPU inference only.

To get started, clone the roop repository and create an `app.py` file. 

we will call the core module in app.py using the following code:

```
# -* coding:UTF-8 -*
# !/usr/bin/env python
import numpy as np
import gradio as gr
import roop.globals
from roop.core import (
    start,
    decode_execution_providers,
    suggest_max_memory,
    suggest_execution_threads,
)
from roop.processors.frame.core import get_frame_processors_modules
from roop.utilities import normalize_output_path
import os
from PIL import Image


def swap_face(source_file, target_file):

    source_path = "input.jpg"
    target_path = "target.jpg"

    source_image = Image.fromarray(source_file)
    source_image.save(source_path)
    target_image = Image.fromarray(target_file)
    target_image.save(target_path)

    print("source_path: ", source_path)
    print("target_path: ", target_path)

    roop.globals.source_path = source_path
    roop.globals.target_path = target_path
    output_path = "output.jpg"
    roop.globals.output_path = normalize_output_path(
        roop.globals.source_path, roop.globals.target_path, output_path
    )
    roop.globals.frame_processors = ["face_swapper"]
    roop.globals.headless = True
    roop.globals.keep_fps = True
    roop.globals.keep_audio = True
    roop.globals.keep_frames = False
    roop.globals.many_faces = False
    roop.globals.video_encoder = "libx264"
    roop.globals.video_quality = 18
    roop.globals.max_memory = suggest_max_memory()
    roop.globals.execution_providers = decode_execution_providers(["cpu"])
    roop.globals.execution_threads = suggest_execution_threads()

    print(
        "start process",
        roop.globals.source_path,
        roop.globals.target_path,
        roop.globals.output_path,
    )

    for frame_processor in get_frame_processors_modules(
        roop.globals.frame_processors
    ):
        if not frame_processor.pre_check():
            return

    start()
    return output_path


app = gr.Interface(
    fn=swap_face, inputs=[gr.Image(), gr.Image()], outputs="image"
)
app.launch()
```

We will use Gradio to design the user interface for our program. The good news is that Hugging Face natively supports Gradio, so we don't need to import any additional libraries.

Once you have written the program, you can push it to your Hugging Face space. After the push, the program will be automatically deployed. All you need to do is wait for the deployment to complete, and then you can start using the one-click face swapping software.

Remember, the deployment process is hassle-free and user-friendly, allowing you to focus on the exciting applications of your software.

![huggingface-deployment](/images/roop-gui.png)
