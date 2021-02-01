# Preparing the map

## Adding HorseExtension files

Unzip this to a folder called HorseExtension in the main folder of your mod map. A lot of later steps depend on having these files there.

## HeightTypes and add multi-terrain angle

Before adding new fruits to the map, the maximum heightTypes needs to be increased to 128, and maximum fruitTypes needs to increased as well.

### Converting GDMs to PNGs

We'll add multi-terrain angle support at the same time as increasing the heightTypes because they both use the same ModMap.lua file.

Find cultivator_density.gdm and terrainDetailHeight_density.gdm (generally under maps/mapDE or maps/mapUS or similar folder). You'll need the GRLE converter from the Downloads section of https://gdn.giants-software.com (registration required). The easiest way to use this is to have the maps/mapXX folder and the unzipped GRLE folder open side by side and drag the GDM files onto convert.cmd.

After using the GRLE converter, you should have cultivator_density.png, and terrainDetailHeight_density.png files (note the PNG extension) in the same folder as the original GDMs.

Delete the original cultivator_density.gdm and terrainDetailHeight_density.gdm files. This is important. The conversion will fail if these files still exist.

### Editing the map's i3d file

Find the main i3d file for the map (e.g. maps/map.i3d, maps/mapDE.i3d, or maps/mapUS.i3d, or sometimes in the root folder instead of maps/). Open it in a text editor like Notepad++, not in the Giant's Editor.

#### Using the converted PNGs

Find the line
```
<File fileId="206" filename="map/cultivator_density.gdm"/>
```
and change it to
```
<File fileId="206" filename="map/cultivator_density.png"/>
```
Then find the line
```
<File fileId="206" filename="map/cultivator_density.gdm"/>
```
and change it to
```
<File fileId="206" filename="map/cultivator_density.png"/>
```

#### New shaders to support increased heightTypes and terrain angles

To use the new shader files included find the line
```
<File fileId="217" filename="$data/shaders/groundHeightShader.xml"/>
```
and change the filename to
```
<File fileId="217" filename="../HorseExtension/shaders/groundHeightShader.xml"/>
```
then find
```
<File fileId="215" filename="$data/shaders/groundShader.xml"/>
```
and change the filename to
```
<File fileId="215" filename="../HorseExtension/shaders/groundShader.xml"/>
```
The fileId numbers might be different. Ignore them, don't change them.

#### Change the heightType parameters

Find the line that starts `<DetailLayer name="terrainDetailHeight"` and change:

- `numDensityMapChannels="11"` to `numDensityMapChannels="13"`
- `compressionChannels="5"` to `compressionChannels="7"` (or add `compressionChannels="7"` after the numDensityMapChannels attribute if compressionChannels isn't there)
- `combinedValuesChannels="0 5 0;5 6 0"` to `combinedValuesChannels="0 7 0;7 6 0"`
- `heightFirstChannel="5"` to `heightFirstChannel="7"`

#### Change the terrain angle parameters

Find the line that starts `<DetailLayer name="terrainDetail"` and change:

- `numDensityMapChannels="13"` to `numDensityMapChannels="16"` (for 3 more angle channels)
- `combinedValuesChannels="0 3 0;6 2 0;3 3 1"` to `combinedValuesChannels="0 3 0;6 5 0;3 3 1"` (yes, just add 3 to the middle number)

### Linking the ModMap.lua

Without some LUA script, this won't work. Luckily, the script is included, but you still need to let the map know to load it. Open the modDesc.xml file in the main folder of your mod, find the line like:
```
<map id="EmptyMap" className="Mission00" filename="$dataS/scripts/missions/mission00.lua" configFilename="xml/map.xml" defaultVehiclesXMLFilename="xml/vehicles.xml" defaultItemsXMLFilename="xml/items.xml">
```
and change:

- `className="Mission00"` to `className="ModMap"`
- `filename="$dataS/scripts/missions/mission00.lua"` to `filename="HorseExtension/scripts/ModMap.lua"`

### Generating the new GDMs

Open the map’s i3d file in the Giants Editor. In the console you might see two lines saying `DensityMap: failed to load image.`. These are fine, but any other errors need to be dealt with.

Save the map, without changing anything. Wait for confirmation from the editor that it's complete, then close the editor. This should create new `cultivator_density.gdm` and `terrainDetailHeight_density.gdm` files in the same place they were before. Check they've been created, then open the i3d in a text editor again, change:
```
<File fileId="206" filename="map/cultivator_density.png"/>
...
<File fileId="216" filename="map/terrainDetailHeight_density.png"/>
```
back to
```
<File fileId="206" filename="map/cultivator_density.gdm"/>
...
<File fileId="216" filename="map/terrainDetailHeight_density.gdm"/>
```
and delete the PNG files you created.

The map should be usable now. Try loading it with several mods that add heightTypes at the same time (seasons, strawHarvest, etc) to make sure you don't get an error for too many height types. Then get a plow and do some donuts on a field to check if more terrain angles are supported (there will still be some tearing in the texture, but it won't change 45 degrees at a time).

# New fruits

## More map preparation

By default FS19 maps don't support enough fruitTypes, so we have to fix that first.

### GDM conversion

Like the heightTypes and MTA support, find `fruit_density.gdm`, convert it to a PNG with the GRLE converter, make sure the new `fruit_density.png` file is in the same directory as the original GDM file, and DELETE the original `fruit_density.gdm`.

### Editing map.i3d

Open the map's main i3d file in a text editor.

Change
```
<File fileId="219" filename="map/fruit_density.gdm"/>
```
to
```
<File fileId="219" filename="map/fruit_density.png"/>
```

Find the FoliageMultiLayer element and change
```
<FoliageMultiLayer densityMapId="219" numChannels="10" numTypeIndexChannels="5" compressionChannels="5">
```
to
```
<FoliageMultiLayer densityMapId="219" numChannels="12" numTypeIndexChannels="6" compressionChannels="6">
```
adding 2 to numChannels, and 1 to each of numTypeIndexChannels and compressionChannels.

### Generating new GDMs

Open the map's main i3d in Giants Editor. Again, `DensityMap: failed to load image.` is fine, but other errors are a problem. Save the map, wait for confirmation, and close the Giants Editor.

Make sure a new `fruit_density.gdm` file was generated, DELETE the converted `fruit_density.png`, and use a text editor to change
```
<File fileId="219" filename="map/fruit_density.png"/>
```
back to
```
<File fileId="219" filename="map/fruit_density.gdm"/>
```
## Linking HorseExtension files

This stage is a lot of copying. In the HorseExtension support folder you'll find all the things you need to set up ALMOST everything else.

### XML additions

There are several files with entries you need to copy to your map's files. The comments in them tell you where everything goes, and some things to watch out for.

`map_i3d_additions.xml` contains entries you need to add to the map's main i3d file. The comments tell you where. These entries add the new crops to the map.

`map_xml_additions.xml` contains the entries you need to add to the map's main xml file. This is normally in the same folder as the main i3d, with the same name, except for a .xml extension. If you're not sure where it is, look in modDesc.xml for the `map` element. It will have an attribute like `configFilename="maps/xmlLua/mapDE.xml"` that tells you where the file is. These entries link up all the visual stuff like displaying crops on the ground, in trailers, what they look like being cut and unloaded and baled. They also link the new animal feeding system.

`modDesc_additions.xml` contains the entries you need to add to your map's `modDesc.xml`. This adds scripts like FruitDestruction 2.0, and CCM support with `FillTypeMover.lua`, support for new crop baling systems, and vehicles for harvesting the new crops. There are also balers, but I've never needed them since MaizePlus adds support to the base game balers (will discuss with the Farming Agency people).

`sellingStation_additions.xml` contains examples for adding support for the new crops to the BGA and sellpoints so that you can sell the new crops. You might want to chop and change this depending on what makes sense for your map. Maybe you want to add rye to a bakery sellpoint but not onions, field_grass, or miscanthus. Maybe you want to sell carrots and onions to a supermarket. Maybe your map has separate sellpoints for grains and root crops.

### MaizePlus support files

Copy the `maizePlus` folder from `HorseExtension/support` to the main folder of your map. Edit the animalFoodAdditions.xml file inside if you want to change the way animal feeding works with MaizePlus.

### Seasons

The `support/seasons` folder contains an example GEO that supports the new crops. You can copy it to your map's main folder, then edit it for the environment you want. There are also MaizePlus GEO mods starting to appear on ModHub if you want more inspiration.

# Crop storage

## Silos

Okay, this is difficult. Most silos don't support the new grain crops. Multifruit silos usually support *everything*, including root crops, woodchips, and other things that don't make sense in a grain silo. Giants actually fucked this one up, because they have a fillTypeCategory called FARMSILO that lists things that make sense to go into a silo, but they don't use it and manually list what silos support.

If the map you're on has custom silos (e.g. in a placeables folder), you can add support yourself.

Find the specs element
```
<specs>
    <capacity>100000</capacity>
    <fillTypes>wheat barley canola maize oat sunflower soybean</fillTypes>
</specs>
```
and add the new crops
```
<specs>
    <capacity>100000</capacity>
    <fillTypes>wheat barley canola maize oat sunflower soybean rye triticale spelt field_grass</fillTypes>
</specs>
```
then find the `storages` element
```
<storages>
    <storage capacityPerFillType="100000" fillTypes="wheat barley canola maize oat sunflower soybean" node="0|2"/>
</storages>
```
and add the new crops there too
```
<storages>
    <storage capacityPerFillType="100000" fillTypes="wheat barley canola maize oat sunflower soybean rye triticale spelt field_grass" node="0|2"/>
</storages>
```

## Haylofts

I'm pretty sure haylofts from Giants actually use the HAYLOFT fillTypeCategory and should work since HorseExtension adds the new crops to that category. There are a few details I still need to work out. Will confer with F/A.

# HorseExtension additions to animal pens

This is something I haven't added. Having things like horses and chickens produce manure requires changes to placeables that are beyond this tutorial. Hopefully some are released as placeable mods instead of being added to maps individually.