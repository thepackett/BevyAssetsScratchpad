## What problem does this solve or what need does it fill?
Currently bevy_asset is geared toward loading and processing assets.
The current pipeline for loading assets looks like (Assuming a custom implementation for everything is needed):
## Bevy_Asset Usage Graph
Definitions:
- AssetSource: A source that bytes can be extracted from. E.g. filesystem, remote, embedded, etc.
- AssetReader: Translates from Source to byte data.
- AssetWriter: Translates from byte data to Source.
- AssetLoader: Translates from bytes to asset.
- AssetSaver: Translates from asset to bytes, possibly also processing the asset.
- AssetServer: Used to load assets.
- AssetProcessor: A background processer that loads all assets from a "source" AssetSource, processes them, and then saves them to a "destination" AssetSource.
  - Meta File: Produced when assets are processed and saved in the same location as the source asset they correspond to.

Using bevy's asset system involves setting up your app (top subgraph starting with `App`), your types (lower left subgraph), and then using the asset server (lower right subgraph).
```mermaid
graph TD
subgraph AppSetup[App Setup]
  direction LR
  app -->|call| init_asset
  app -->|call| init_asset_loader
  app -->|call| register_asset_source
  app -.->|call| register_asset_processor
  app -.->|call| set_default_asset_processor
  register_asset_source -->|id is| asset_source_id
  register_asset_source -->|source is| asset_source_builder
  register_asset_source -->|has requirement| register_asset_source_note_requirement
  register_asset_source -->|has side effect| register_asset_source_note_side_effect
  register_asset_processor -.->|extension is| asset_processor_extension
  register_asset_processor -.->|for P| processor_note
  set_default_asset_processor -.->|extension is| asset_processor_extension
  set_default_asset_processor -.->|has side effect| set_default_asset_processor_side_effect
  set_default_asset_processor -.->|for P| processor_note
  app["App"]
  init_asset["app.init_asset::&lt;CustomAsset&gt;()"]
  init_asset_loader["app.init_asset_loader::&lt;CustomAssetLoader&gt;()"]
  register_asset_source["app.register_asset_source(id, source)"]
  register_asset_processor["app.register_asset_processor::&lt;P: Process&gt;(extension)"]
  set_default_asset_processor["app.set_default_asset_processor::&lt;P: Process&gt;(extension)"]
  set_default_asset_processor_side_effect["Replaces any existing asset processor for this extension with the given asset processor."]
  register_asset_source_note_requirement["Must be called before adding the AssetPlugin"]
  register_asset_source_note_side_effect["If an asset source already exists with the given Id, it is replaced."]
  asset_processor_extension["A &str that represents a file extension.\nE.g. &quot;cstm&quot; would apply to files with the &quot;.cstm&quot; extension"]
  processor_note["Although you can implement Process on your own type,\nit is recommended to use LoadAndSave&lt;L: AssetLoader, S: AssetSaver&lt;Asset = L::Asset&gt;&gt;"]
  subgraph AssetSourceId
    asset_source_id -->|with variant| asset_source_id_default
    asset_source_id -->|with variant| asset_source_id_name
    asset_source_id_default --> asset_source_id_default_explanation
    asset_source_id_name --> asset_source_id_name_explanation
    asset_source_id["Enum"]
    asset_source_id_default["AssetSourceId::Default"]
    asset_source_id_name["AssetSourceId::Name"]
    asset_source_id_default_explanation["When an AssetPath does not provide a source, this source is used.\nE.g. &quot;/path/to/asset.cstm&quot;"]
    asset_source_id_name_explanation["When an AssetPath does provide a source, this source is used.\nE.g. &quot;custom://path/to/asset.cstm&quot;"]
  end
  subgraph AssetSourceBuilder
    asset_source_builder_get -->|gets you| asset_source_builder
    asset_source_builder -->|has method| asset_source_builder_with_reader
    asset_source_builder -->|has method| asset_source_builder_with_writer
    asset_source_builder -->|has method| asset_source_builder_with_watcher
    asset_source_builder -->|has method| asset_source_builder_with_processed_reader
    asset_source_builder -->|has method| asset_source_builder_with_processed_writer
    asset_source_builder -->|has method| asset_source_builder_with_processed_watcher
    asset_source_builder -->|has method| asset_source_builder_with_watcher_warning
    asset_source_builder -->|has method| asset_source_builder_with_processed_watcher_warning
    asset_source_builder -->|has method| asset_source_builder_platform_default

    asset_source_builder["AssetSourceBuilder"]
    asset_source_builder_get["AssetSource::build()"]
    asset_source_builder_with_reader["with_reader(...)"]
    asset_source_builder_with_writer["with_writer(...)"]
    asset_source_builder_with_watcher["with_watcher(...)"]
    asset_source_builder_with_processed_reader["with_processed_reader(...)"]
    asset_source_builder_with_processed_writer["with_processed_writer(...)"]
    asset_source_builder_with_processed_watcher["with_processed_watcher(...)"]
    asset_source_builder_with_watcher_warning["with_watcher_warning(...)"]
    asset_source_builder_with_processed_watcher_warning["with_processed_watcher_warning(...)"]
    asset_source_builder_platform_default["platform_default(...)"]
  end
end
subgraph TypeSetup[Type Setup]
  direction LR
  custom_asset_loader -->|impl| asset_loader
  custom_asset_saver -->|impl| asset_saver
  custom_asset -->|impl/derive| asset
  custom_asset_reader -->|impl| asset_reader
  custom_asset_writer -->|impl| asset_writer
  custom_asset_loader["CustomAssetLoader"]
  custom_asset_saver["CustomAssetSaver"]
  custom_asset_reader["CustomAssetReader"]
  custom_asset_writer["CustomAssetWriter"]
  asset_loader["AssetLoader"]
  asset_saver["AssetSaver"]
  asset_reader["AssetReader"]
  asset_writer["AssetWriter"]
  custom_asset["CustomAsset"]
  asset["Asset"]
end
subgraph AssetServer[AssetServer]
  direction LR
  world -->|gets you| asset_server
  query -->|gets you| asset_server
  asset_server -->|has method| asset_server_load
  world["World: world.get_resource::&lt;AssetServer&gt();"]
  query["Query: Res&lt;AssetServer&gt;"]
  asset_server["AssetServer"]
  asset_server_load["asset_server.load(&quot;an_asset.cstm&quot;)"]
end
AppSetup ~~~ TypeSetup
AppSetup ~~~ AssetServer
classDef Green stroke:#BDFFA4,stroke-width:4px;
class app Green;
```
```mermaid
graph TD
asset_server["AssetServer"]
asset_processor["AssetProcessor"]
```

TODO:
- Adding the asset plugin is a requirement.
- Describe AssetProcessor behavior

Notes:
What information needs to be conveyed?
- How a user *uses* these systems, which should be visually distinct from
- How these systems work internally, at least to a high level approximation

This system is increadibly flexible, but it is worth noting that saving is unnecessarily coupled to processing.

## What solution would you like?

I would like to split processing and saving into two distinct parts. The current Saver will only be responsible for converting an asset into bytes, and a new Processor will be introduced which is responsible for actually operating on an asset.
So for instance, LoadAndSave<L: Loader, S: Saver> processor would become LoadAndSave<L: Loader, P: Processor, S: Saver>. 

With Savers decoupled from Processors, it would allow us to expose saving functions through the AssetServer that can utilize registered Savers. This API may look something like save::<A: Asset>(path: Path, asset: A). This is primarily useful for saving DynamicScenes, but could also be used for saving things like screenshots.

## Additional context

Any other information you would like to add such as related previous work,
screenshots, benchmarks, etc.
