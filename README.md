## Bevy_Asset High Level Overview
### Definitions:
- **AssetSource (Trait)**: A source that bytes can be extracted from and/or written to. E.g. filesystem, remote, embedded, etc.
- **AssetReader (Trait)**: Translates from Source data to byte data.
- **AssetWriter (Trait)**: Translates from byte data to Source data.
- **AssetLoader (Trait)**: Translates from bytes to an Asset.
- **AssetSaver (Trait)**: Translates from an Asset to bytes, possibly also processing the Asset.
- **AssetServer (Resource)**: Used to load assets.
- **AssetProcessor (Resource)**: A background processer that loads all assets from a "source" AssetSource, processes them, and then saves them to a "destination" AssetSource.
  - **Meta File**: Produced when Assets are processed and saved. Stored in the same location as the Asset they correspond to.

### Asset Loading:
```mermaid
graph LR
asset_source -->|source data| asset_reader
asset_reader -->|byte data| asset_loader
asset_loader --> asset
asset["Asset"]
asset_source["AssetSource"]
asset_reader["AssetReader:\nTranslates source data into byte data"]
asset_loader["AssetLoader:\nTranslates byte data into an Asset"]
```
### Asset Saving:
```mermaid
graph RL
asset -->|read by| asset_saver
asset_saver -->|byte data| asset_writer
asset_writer -->|source data| asset_source
asset["Asset"]
asset_source["AssetSource"]
asset_writer["AssetWriter:\nTranslates byte data into source data"]
asset_saver["AssetSaver:\nTranslates an Asset to bytes, possibly processing it"]
```
### Asset Processing:
```mermaid
graph TD
asset_reader -->|byte data| process_trait
process_trait -->|processed byte data| asset_writer
asset_loader -->|loaded asset| load_save_processor
load_save_processor -->|processed asset| asset_saver
asset_reader["AssetReader"]
asset_writer["AssetWriter"]
asset_loader["AssetLoader"]
asset_saver["AssetSaver"]
process_trait["impl Process"]
load_save_processor["LoadAndSave&lt;L: AssetLoader, S: AssetSaver&gt;"]
```

## Bevy_Asset Usage Overview (Assuming a custom implementation is needed for everything)
### Type Setup
Using bevy's asset system first involves setting up the types that your Asset needs:
```mermaid
graph LR
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
```
### App setup
Secondly, you must set up your App. In this diagram, start from the "App" node in the lower left, and call the necessary methods for your custom asset implementation:
```mermaid
graph LR
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
```
### AssetServer Usage
Lastly, to load your custom asset you can use the AssetServer:
```mermaid
graph LR
world -->|gets you| asset_server
  query -->|gets you| asset_server
  asset_server -->|has method| asset_server_load
  world["World: world.get_resource::&lt;AssetServer&gt();"]
  query["Query: Res&lt;AssetServer&gt;"]
  asset_server["AssetServer"]
  asset_server_load["asset_server.load(&quot;an_asset.cstm&quot;)"]
```

## Bevy Asset Internals
### AssetServer
- 

### AssetProcessor
- contains an exclusive AssetServer which is configured for asset processing requirements
  - Such as?
- contains 'data' which represents the state of the AssetProcessor. This state includes data on
  - what processors and default processors are added
  - the current state of the AssetProcessor (Initializing, Processing, or Finished)
  - asset sources
  - Senders and recievers for initialization and finished "events" that are emitted when the AssetProcessor has its state changed.
  - A log which tracks the state of asset processing to ensure that processing operations are atomic.
- When the AssetProcessor is started, it immediately starts processing all configures assets. Once it is done it blocks on listening for source change events.
  - Processing all files involves 
  - When it recieves a source change event, it takes action based on the event recieved. I.e. if an asset added event is received, it processes that asset.

### Meta files
- Files that communicate to the AssetServer or AssetProcessor how an asset should be handled.
- Contains:
  - AssetMetaMinimal
    - The bare minimum that a .meta file may contain.
      - Lists meta information, such as meta format version and an:
      - AssetActionMinimal, which is an enum with variants Load, Process, and Ignore. Each variant stores the required information for the action it represents.
- Meta files are generated:
  - When processing assets, if an asset does not have a meta file then one is generated from the default meta file for the default processor for an asset's file extension if it exits, otherwise from the loader for that file extension.
