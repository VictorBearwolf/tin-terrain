# TIN Terrain

[![Build Status](https://travis-ci.com/heremaps/tin-terrain.svg?branch=master)](https://travis-ci.com/heremaps/tin-terrain)

TIN Terrain is a command-line tool for converting heightmaps presented in GeoTIFF format into tiled optimized meshes (Triangulated Irregular Network) with different levels of details.

Check out [heremaps/quantized-mesh-viewer](https://github.com/heremaps/quantized-mesh-viewer) for examples of rendering output in [Cesium.js](https://github.com/heremaps/quantized-mesh-viewer/blob/master/src/map/surface-provider.js) and [Three.js](https://github.com/heremaps/quantized-mesh-viewer/blob/master/src/tile/index.js).

Note: This is experimental code, expect changes.

![dem2tin GeoTIFF to mesh on USGS craterlake DEM](docs/resources/dem2tin.jpg)

## Features

* Takes GeoTIFF with heightmap as an input
* Transforms heightmap into a TIN mesh with a given max-error parameter and outputs into `.obj` format
* Transforms heightmap into tiled TIN mesh for a given zoom range and outputs tiled pyramid into [quantized-mesh-1.0](https://github.com/AnalyticalGraphicsInc/quantized-mesh) terrain format

## Let us know what you think!

You can help us a lot to prioritize issues in tin-terrain if you submit a [short anonymous survey](https://survey.research-feedback.com/jfe/form/SV_6qVft7VrQfwsDml)

## Installation

You can build and run this tool either with Docker or directly on your system. Building with Docker is easier and works on macOS, Linux, and Windows.

Detailed instructions for each platform are provided below.


### Building with Docker

The provided Dockerfile builds the TIN Terrain executable on Ubuntu Linux.

To build the container, execute:

```
./build-docker.sh
```

To run TIN Terrain from Docker, type e.g. 

```
docker run -v [local data directory]:/data:cached --name tin-terrain --rm -i -t tin-terrain tin-terrain ...
```

where `[local data directory]` is the folder that contains your DEM files and will receive the output files. This will be mapped to `/data` in the Docker instance, so you should use:

```
docker run -v [local data directory]:/data:cached --name tin-terrain --rm -i -t tin-terrain tin-terrain dem2tin --input /data/... --output /data/... 
```

Alternatively, use 

```
docker run -v [local data directory]:/data:cached --name tin-terrain --rm -i -t tin-terrain bash 
```
    
to open an interactive shell which lets you execute `tin-terrain` and access `/data`. Any files not stored in `/data` will be lost when closing the session.


### Prerequisites & Dependencies

If you choose to compile and run "TIN Terrain" directly on your system, please note that the following packages must be installed on your system:
* The C++ standard library from the compiler (tested with `clang` `9.1.0` and `gcc` `7.3.0`)
* Boost (BSL-1.0) `program_options`, `filesystem`, `system` (tested with version `1.67`)
* GDAL (MIT) `gdal`, `gdal_priv`, `cpl_conv` (tested with version `2.3`)

TIN Terrain also downloads some source code of 3rd party libraries during the CMake configure phase:
* [libfmt](https://github.com/fmtlib/fmt) (BSD-2-Clause)
* [glm](https://github.com/g-truc/glm) (MIT)
* [Catch2](https://github.com/catchorg/Catch2) (BSL-1.0)

You can disable this behaviour by passing the `-DTNTN_DOWNLOAD_DEPS=OFF` option to CMake when generating the project/makefiles.
If you do so, you have to download dependencies yourself and also pass the `TNTN_LIBGLM_SOURCE_DIR` and `TNTN_LIBFMT_SOURCE_DIR` variables to CMake
as well as `TNTN_CATCH2_SOURCE_DIR` if you want to run the tests.

See [download-deps.cmake](download-deps.cmake) for detailed version info.


### Building on macOS

1. Install dependencies, preferably with homebrew:
    ```
    brew install boost
    brew install cmake
    brew install gdal
    ```
2. Create an XCode project:
    ```
    mkdir build-cmake-xcode
    cd build-cmake-xcode
    cmake -GXcode path/to/sourcecode/
    open tin-terrain.xcodeproj
    ```
3. Build the TIN Terrain target.

The resulting binaries will be in the Debug/Release subdirectory.

To run the tests, build and run the tntn-tests target.

#### Possible linking issues with GDAL

If another version of GDAL is present on your machine, the FindGDAL.cmake provided with cmake might not be able
to properly detect the newest version of GDAL you installed through homebrew (or any other prefered method).

Whether this is the case, can be easily detected, if the linking step in the compilation errors on linking GDAL.

Therefore you might need to guide cmake to the right location of the GDAL:

```
cmake -GXcode -DGDAL_LIBRARY=PATH/TO/GDAL/LIB path/to/sourcecode/
```
e.g
```
cmake -GXcode -DGDAL_LIBRARY=/usr/local/Cellar/gdal/2.3.1_2/lib/libgdal.dylib path/to/sourcecode/
```


### Building on Linux

1. Install dependencies, e.g. on Ubuntu:
    ```
    apt-get install build-essential cmake libboost-all-dev libgdal-dev
    ```
2. Create Makefile:
    ```
    mkdir build-cmake-release
    cd build-cmake-release
    cmake -DCMAKE_BUILD_TYPE=Release path/to/sourcecode/
    ```
3. Build the `tin-terrain` target
    ```
    VERBOSE=1 make tin-terrain
    ```

The resulting binary should then be ready.
To run the tests, build and run the tntn-tests target:

1. Recreate Makefile (and set TNTN_TEST=ON):
    ```
    cd build-cmake-release
    cmake -DTNTN_TEST=ON -DCMAKE_BUILD_TYPE=Debug path/to/sourcecode/
    ```
2. Build the `tntn-tests` target
    ```
    VERBOSE=1 make tntn-tests
    ```

## Usage

The `tin-terrain` command-line tool has a few subcommands. You can run `tin-terrain --help` to see all available subcommands.

```
$ tin-terrain --help
usage:
  tin-terrain [OPTION]... <subcommand> ...

Global Options:
  -h [ --help ]               print this help message
  --log arg (=stdout)         diagnostics output/log target, can be stdout,
                              stderr, or none
  -v [ --verbose ] [=arg(=1)] be more verbose
  --subcommand arg            command to execute
  --subargs arg               arguments for command

available subcommands:
  dem2tin - convert a DEM into a mesh/tin
  dem2tintiles - convert a DEM into mesh/tin tiles
  benchmark - run all available meshing methods on a given set of input files and produce statistics (performance, error rate)
  version - print version information
```


### Projections

The `tin-terrain` tool requires your datasets to be in the Web Mercator projection (EPSG:3857). If your datasets are not in this projection, you can quite easily reproject your datasets with `gdalwarp`, e.g.:

```
gdalwarp -t_srs EPSG:3857 ned19_n37x75_w122x50_ca_goldengate_2010.img ned19_n37x75_w122x50_ca_goldengate_2010_mercator.tif
```


### Creating a single mesh/TIN

You can use the `dem2tin` subcommand to convert a raster heightmap into a single big mesh/TIN in the OBJ format.

You can see all available options by running `tin-terrain dem2tin --help`.

```
$ tin-terrain dem2tin --help
usage:
  tin-terrain dem2tin [OPTIONS]...

dem2tin options:
  --input arg                 input filename
  --input-format arg (=auto)  input file format, can be any of: auto, asc, xyz,
                              tiff
  --output arg                output filename
  --output-format arg (=auto) output file format, can be any of: auto, obj,
                              off, terrain (quantized mesh), json/geojson
  --method arg (=terra)       meshing method, valid values are: terra, zemlya and dense
  --max-error arg             (terra & zemlya) maximum geometric error
  --step arg (=1)		      (dense) grid spacing in pixels


methods:
  terra     - implements a delaunay based triangulation with point selection using a fast greedy insert mechnism
    reference: Garland, Michael, and Paul S. Heckbert. "Fast polygonal approximation of terrains and height fields." (1995).
    paper: https://mgarland.org/papers/scape.pdf
    and http://mgarland.org/archive/cmu/scape/terra.html
  zemlya    - hierarchical greedy insertion
    reference: Zheng, Xianwei, et al. "A VIRTUAL GLOBE-BASED MULTI-RESOLUTION TIN SURFACE MODELING AND VISUALIZETION METHOD." International Archives of the Photogrammetry, Remote Sensing & Spatial Information Sciences 41 (2016).
    paper: https://www.int-arch-photogramm-remote-sens-spatial-inf-sci.net/XLI-B2/459/2016/isprs-archives-XLI-B2-459-2016.pdf
  dense     - generates a simple mesh grid from the raster input by placing one vertex per pixel
```

For example, you can run the following command to convert "ned19_n37x75_w122x50_ca_goldengate_2010_mercator.tif" to a single big mesh, using a `max-error` parameter of 2.0 meters:

```
tin-terrain dem2tin --input /data/ned19_n37x75_w122x50_ca_goldengate_2010_mercator.tif --output /data/terrain.obj --max-error 2.0
```

After running this command, you should see a "terrain.obj" output file in the same directory. This OBJ file contains the generated mesh.

The `max-error` parameter specifies the vertical error allowance in meters, so a smaller `max-error` parameter results in more triangles in the output mesh, better mesh quality, and longer running time.


### Creating a pyramid of mesh/TIN tiles

The `dem2tintiles` subcommand takes a raster heightmap as the input, and creates tiled TIN mesh for a given zoom range. The output tiles can be either in the OBJ format or in the [quantized-mesh-1.0](https://github.com/AnalyticalGraphicsInc/quantized-mesh) terrain format.

You can see all available options by running `tin-terrain dem2tintiles --help`.

```
$ tin-terrain dem2tintiles --help
usage:
  tin-terrain dem2tintiles [OPTION]...

dem2tintiles options:
  --input arg                    input filename
  --output-dir arg (=./output)
  --max-zoom arg (=-1)           maximum zoom level to generate tiles for. will
                                 guesstimate from resolution if not provided.
  --min-zoom arg (=-1)           minimum zoom level to generate tiles for will
                                 guesstimate from resolution if not provided.
  --max-error arg                (terra or zemlya) max error when using
  --step arg (=1)            	 (dense) grid spacing in pixels
  --output-format arg (=terrain) output tiles in terrain (quantized mesh) or
                                 obj
  --method arg (=terra)          meshing algorithm. one of: terra, zemlya or dense
```


For example, you can run the following command to convert "ned19_n37x75_w122x50_ca_goldengate_2010_mercator.tif" to a pyramid of mesh tiles.

```
tin-terrain dem2tintiles --input /data/ned19_n37x75_w122x50_ca_goldengate_2010_mercator.tif --output-dir /data/output --min-zoom 5 --max-zoom 14 --output-format=terrain --max-error 2.0
```

When this command finishes running, you should see an output folder containing all the mesh tiles, organized into a pyramid of subfolders.

    ├── 10
    │   ├── 163
    │   │   ├── 627.terrain
    │   │   └── 628.terrain
    │   └── 164
    │       ├── 627.terrain
    │       └── 628.terrain
    ...

The folder structure follows the map tile convention: `Z/X/Y.terrain`.

These mesh tiles can then be easily served from a webserver and be consumed by frontend applications for purposes such as terrain visualization.

### Sample Datasets

When you enable the `TNTN_TEST` and `TNTN_DOWNLOAD_DEPS` options in the CMake configuration, a few sample datasets will be downloaded into the `${CMAKE_SOURCE_DIR}/3rdparty/` folder.

For example, you will find the crater lake dataset in the `${CMAKE_SOURCE_DIR}/3rdparty/craterlake` folder. This dataset is created and maintained by the U.S. Geological Survey (USGS) and can be downloaded from <http://oe.oregonexplorer.info/craterlake/>.

To generate a mesh from this dataset, you need to reproject it to the Web Mercator projection first, using the `gdalwarp` command-line tool which comes with your GDAL installation:

```sh
gdalwarp -t_srs EPSG:3857 -r cubic -of GTiff -ot Float32 dems_10m.dem dems_10m.tif
```

Then you can run `tin-terrain` on this reprojected GeoTIFF file to create a mesh:

```sh
./tin-terrain dem2tin --input dems_10m.tif --output terrain.obj --max-error 2.0 --method terra
```

## License

Copyright (C) 2018 HERE Europe B.V.

See the [LICENSE](LICENSE) file in the root of this project for license details.

## Papers

Some algorithms are based on ideas from:

* [Garland, Michael, and Paul S. Heckbert. "Fast polygonal approximation of terrains and height fields." (1995).](https://mgarland.org/papers/scape.pdf)
* [Zheng, Xianwei, et al. "A VIRTUAL GLOBE-BASED MULTI-RESOLUTION TIN SURFACE MODELING AND VISUALIZETION METHOD." International Archives of the Photogrammetry, Remote Sensing & Spatial Information Sciences 41 (2016).](https://www.int-arch-photogramm-remote-sens-spatial-inf-sci.net/XLI-B2/459/2016/isprs-archives-XLI-B2-459-2016.pdf)
