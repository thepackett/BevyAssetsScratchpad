# GLTF Overview
- Graphics Library Transmission Format. Made by Khronos (OpenGL, Vulkan, etc).
- .gltf:
  - json file with "pointers" to dense binary resources such as meshes, materials, etc.
- .glb:
  - All in one gltf file "header" with dense binary resources appended. Limited to 4 GB.
- Well defined specification for what is included: https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html
- Additional system for:
  - Extensions:
    - Optional additions to the gltf format that are appended to any json objects. Can contain arbitrary data. Examples: Index of Refraction, Emmisive strength, etc.
Extensions must be declared at the beginning of the file as being used or required. Extensions that are used but not supported are ignored. If an extension is required but not supported, then the file must fail to load.
  - Extras:
    - Very similar to extensions, these may contain arbitrary data, but do not need to be declared at the beginning of the file. Typically used to contain application specific data.

Problem: 
- Not all extensions "follow the rules". While appending data without affecting the base gltf data is great in most situations, some extensions, such as Draco for mesh compression, have a dilema. Draco works entirely by appending data like other extensions,
however storing compressed mesh data alongside uncompressed mesh data defeats the purpose of the extension. Thus when using Draco, some fields that are usually required in the core GlTF spec become optional, breaking the core idea that unsupported extensions can be ignored.
  - After reading through [every officially ratified glTF extension](https://github.com/KhronosGroup/glTF/blob/main/extensions/README.md), only `KHR_draco_mesh_compression`, `KHR_mesh_quantization`, `KHR_texture_basisu`, `KHR_texture_transform`, `EXT_meshopt_compression`, and `EXT_texture_webp` can create noncompliant gltf files.
    -  `KHR_draco_mesh_compression`, `KHR_texture_basisu`, `KHR_texture_transform`, `EXT_meshopt_compression`, and `EXT_texture_webp` all follow the pattern of making some part of the original gltf specification optional.
    -  `KHR_mesh_quantization` is unique in that it allows meshes to be represented by more data types than a float, which the original spec requires. It is also unique in that it is not possible to specify a fallback, and so if this extension is used it must be required.
- As a game engine, bevy must support both "compliant" gltf assets and "non-compliant" gltf assets.
  - As shown above, this mean we must be able to support normally required fields being optional, as well as potentially having additional data type representation other than what is specified in the core gltf spec.

# Bevy_Gltf
## Current Setup
### gltf-rs
Used for parsing gltf json into a friendlier rust format, gltf validation, gltf helper functions for loading binary resources, etc.

### Bevy_Gltf
Uses gltf to parse information to create our internal representation of a Gltf Asset.

## Issues / Missing features
We are currently missing support for extensions.
