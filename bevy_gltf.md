# GlTF Overview
Graphics Library Transmission Format. Made by Khronos (OpenGL, Vulkan, etc).
.gltf: json file with "pointers" to dense binary resources such as meshes, materials, etc.
.glb: All in one gltf file "header" with dense binary resources appended. Limited to 4 GB.
Well defined specification for what is included: https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html
Additional system for:
Extensions:
Optional additions to the gltf format that are appended to any json objects. Can contain arbitrary data. Examples: Index of Refraction, Emmisive strength, etc.
Extensions must be declared at the beginning of the file as being used or required. Extensions that are used but not supported are ignored. If an extension is required but not supported, then the file must fail to load.

Extras:
Very similar to extensions, these may contain arbitrary data, but do not need to be declared at the beginning of the file. Typically used to contain application specific data.

Problem: Not all extensions "follow the rules". While appending data without affecting the base gltf data is great in most situations, some extensions, such as Draco for mesh compression, have a dilema. Draco works entirely by appending data like other extensions,
however storing compressed mesh data alongside uncompressed mesh data defeats the purpose of the extension. Thus when using Draco, some fields that are usually required in the core GlTF spec become optional, breaking the core idea that unsupported extensions can be ignored.

As a game engine, bevy must support both "conformant" gltf assets and "non-conformant" gltf assets.

# Bevy_Gltf
## Current Setup
### gltf-rs
Used for parsing gltf json into a friendlier rust format, gltf validation, gltf helper functions for loading binary resources, etc.

### Bevy_Glts
Uses gltf to parse information to create our internal representation of a Gltf Asset.

## Issues / Missing features
We are currently missing support for extensions.
