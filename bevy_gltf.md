# GLTF High Level Overview
GLTF stands for Graphics Library Transmission Format, and is an industry standard file format for representing 3D scenes developed by the Khronos Group (known for OpenGL, Vulkan, etc). GLTF files can take two different forms:
- `.gltf` which is a JSON file describing a 3D scene along with resources (.bin, images, etc) either stored externaly in binary/image/etc files, or internally in base64 encoded strings.
- `.glb` which stores the gltf asset (JSON, .bin, images, etc) in a binary blob.
  
In addition to the core GLTF specification (which covers most things like textures, meshes, scenes, etc), GLTF also supports `Extensions`. 

### Extensions
An `Extension` is additional data that can be appended to any JSON object to provide additional functionality. For example, the `KHR_materials_ior` extension appends a single float to a material that represents its Index of Refraction. 

`Extensions` were designed to be an interoperable way to extend GLTF functionality without affecting the core GLTF specification, with unsupported `extensions` simply being ignored. However, some `extensions` do need to affect the core GLTF specification in a non-interoperable way. For example, `EXT_meshopt_compression` which appends pointers to buffers containing compressed mesh data makes the original non-compressed mesh buffer pointer optional (it may be included for interoperability, but it somewhat defeats the purpose to include both compressed and uncompressed data).

All `extensions` used must be listed in the root GLTF object's `extensionsUsed` array. If an extension produces a core GLTF specification non-compliant file, then the extension must be listed in the root GLTF object's `extensionsRequired` array.

### GLTF Specification
The full GLTF specification can be found [here](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html).


# GLTF in Bevy

### How it currently works
Bevy currently uses [gltf-rs](https://github.com/gltf-rs/gltf) to parse a `.gltf` file's bytes into a rust friendly data structure which is then used to fill out our [internal GLTF representation](https://docs.rs/bevy/latest/bevy/gltf/index.html) with data.
Our internal GLTF representation covers the core GLTF specification, as well as `Extras`.

### Problem
Bevy does not currently support any extensions, GLTF compliant or non-compliant, and because of this we are missing out on a large number of very useful GLTF features.

### Solution
Bevy needs a flexible internal GLTF representation, as well as an interface for custom GLTF extensions.

After inspecting [every officially ratified glTF extension](https://github.com/KhronosGroup/glTF/blob/main/extensions/README.md), I found that only `KHR_draco_mesh_compression`, `KHR_mesh_quantization`, `KHR_texture_basisu`, `KHR_texture_transform`, `EXT_meshopt_compression`, and `EXT_texture_webp` can create noncompliant gltf files.
- `KHR_draco_mesh_compression`, `KHR_texture_basisu`, `KHR_texture_transform`, `EXT_meshopt_compression`, and `EXT_texture_webp` all follow the pattern of making some part of the original gltf specification optional.
- `KHR_mesh_quantization` is unique in that it allows mesh data to be represented by other data types than a float, which the original spec requires. It is also unique in that it is not possible to specify a fallback, and so if this extension is used it must be required.

With this in mind, we can come up with some characteristics that our internal GLTF representation must have:
- The internal representation must include all features in the core GLTF specification, but must also:
  - Allow Extensions to mark core GLTF data as optional
  - Allow Extensions to change the data type of some core GLTF data
- 


The internal GLTF representation absolutely needs to be reworked.


Motivating example: An extension would be perfect for representing meshlets in a scene.
