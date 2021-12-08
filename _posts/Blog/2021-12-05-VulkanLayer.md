---
title: "Creating a Vulkan Layer"
date: 2021-12-05T15:34:30-04:00
typora-root-url: ../../

header:
  teaser: /assets/Blog/Decompilation/Teaser.png
  og_image: /assets/Blog/Decompilation/Thumb.png
  image: /assets/Blog/Decompilation/Thumb.png

categories:
  - Blog
tags:
  - Vulkan
  - C
  - Vulkan Layer
  - Low Level
  - Vulkan Loader
  - Computer Graphics
  - GPU 


---
While creating your first Vulkan application, you likely enabled some form of Validation layer, this would ensure that you are using Vulkan correctly. The fantastic part of Vulkan is that this layer isn't built in, in fact you can make your own Vulkan layer; and doing so is a fantastic learning exercise to grown an understanding of the Vulkan loader.

## Debugging The Vulkan Loader
We'll start with a brief introduction into what the Vulkan loader is, and how it works. With Vulkan, you can have multiple different drivers and devices installed into the same system, the loader is responsible for taking your Vulkan calls and passing them through the loaded layers, and eventually onto the correct GPU driver. The Vulkan loader is completley detached from the installed device drivers, so we can actually build the loader from source! This will let us step through the code and get a much better understanding of how things work.

### Building From Source

The repo for the Vulkan loader is hosted on GitHub [here.](https://github.com/KhronosGroup/Vulkan-Loader) In case anything goes wrong, or these build instructions become out of date, then you can check here for a reference. But we'll start by pulling the repo, using a terminal in the location you want to store the loader.
*(Windows devs, in explorer shift right click -> Open Power shell window here)*

```console
git clone https://github.com/KhronosGroup/Vulkan-Loader.git
cd Vulkan-Loader
```
From here, if anything goes wrong, check out the [build instructions](https://github.com/KhronosGroup/Vulkan-Loader/blob/master/BUILD.md) on GitHub. 
Start by pulling all the dependencies of the Vulkan loader, there's a python script to do this for you.

```console
python scripts/update_deps.py
```
Now build in debug mode.
```console
mkdir build 
cd build
cmake -A x64 -DVULKAN_HEADERS_INSTALL_DIR=$(pwd)/../Vulkan-Headers/build/install/ ..
cmake --build . --config Debug
```
The variable being passed to ``VULKAN_HEADERS_INSTALL_DIR`` is an absolute path, it can't be a relative one. ``$(pwd)`` automatically gets transformed into the absolute path of the current directory, although this might be specific to Cygwin and Unix. Base Windows users might have to manually replace ``$(pwd)`` for the absolute path.

### Stepping Into The Loader

Once you've got the Vulkan loader built from source, in debug configuration. The library will now have debug symbols, meaning that you can actually step through it line by line like you would any other program. 

In order to do this, we need to ensure that our application links to our debug version of the loader instead of to the release version installed on your system. You don't want to install the debug version, as it will slow down all your games! 

On Windows this is pretty easy, you just copy and paste the dll to the same folder as the executable, when the executable starts it will prefer to link to the debug version instead of the system version. 

![](/assets/Blog/VulkanLayer/WindowsLoader.png)

On Linux you can override the library search path by setting the ``LD_LIBRARY_PATH`` environment variable to contain the directory that you are keeping the debug loader in.

```bash
export LD_LIBRARY_PATH=/path/to/folder/containing/debug/loader:$LD_LIBRARY_PATH
```

Now when you step click "step into" when highlighting a Vulkan call, we can peak inside the code of the Vulkan loader!

![](/assets/Blog/VulkanLayer/StepInto.png)

## Layer Discovery

The next step in creating our fake Vulkan layer is discovering how on earth the Vulkan loader goes about finding which layers are available to the device? Renderdoc has a really good page that goes into a lot of detail [here.](https://renderdoc.org/vulkan-layer-guide.html) But the end result is that the loader searches in predefined locations, or the directory pointed at by the environment variable ``VK_LAYER_PATH``. Rather than installing our phony layer every time we make a change, we'll use the environment variable to point to the build directory of the layer.

Stepping into the loader we can see that first the implicit layers are loaded, these are the ones installed on your system, if you have something like OBS installed, here's where you'll spot an OBS layer. On windows you can find these layer locations inside the registry. However we're looking for when the explicit layers are discovered. You'll find the code responsible about midway through the function called ``loader_scan_for_layers`` inside of ``loader.c``:
![image-20211208191759956](/assets/Blog/VulkanLayer/FindManifestFiles.png)

You'll notice two things about this function call, first; it's looking for a "manifest file" not the layer itself. Secondly the function is being passed the contents of the `VK_LAYER_PATH` environment variable. Which means the loader is searching for a "manifest file" inside our chosen directory. Let's place a break point here and investigate further. Stepping into this function, the comments tell you exactly what the loader is looking for! A .json file!

![image-20211208193335811](/assets/Blog/VulkanLayer/LayerFinder.png)

So let's place an empty .json file inside the `VK_LAYER_PATH` directory, restart debugging the application and see what happens! This time after `read_data_files_in_search_paths` executes, we can see that the variable `out_files` contains the file path to our .json file!

![image-20211208194638416](/assets/Blog/VulkanLayer/jsonFound.png)

But unfortunately, this is not enough to get our layer to appear in `VkEnummerateInstanceLayerProperties`. What else needs to be done to ensure that our Vulkan layer is shown to the user? Well returning out of `loader_get_data_files` and back into `loader_scan_for_layers`, we can see that the manifest file is parsed, and this is where our layer is getting rejected. So let's actually populate our json file with some reasonable data, instead of being empty. The data I entered was based on the validation layer's json file with all the detail stripped out. 

````json
{
	"file_format_version" : "1.0.0",
	"layer" :{
		"name" : "Fake_Layer",
		"type" : "GLOBAL",
		"library_path": ".\\Fake_Layer.dll",
		"api_version" : "1.0.0",
		"implementation_version": "1",
		"description": "Fake layer for learning",
		"instance_extensions": [],
		"device_extensions": []
	}
}
````

And you might not believe it, but we've successfully tricked the Vulkan loader into thinking we have a fully valid instance layer!
![image-20211208201316333](/assets/Blog/VulkanLayer/LayerEnumerated.png)
