# .root to .gltf exporter

> The following is a tool that can be used to convert .root files that describe a particular geometry to GLTF format, compatible with the [phoenix](https://github.com/HSF/phoenix) framework for visualization

## Quickstart for EIC

The conversion starts from the TGeo/DD4hep ROOT description as produced by the continuous integration pipeline. Download the geometry .root file artifact for the latest successful job with, for example,
```console
wget -O eic.root https://eicweb.phy.anl.gov/EIC/detectors/ecce/-/jobs/artifacts/main/raw/geo/detector_geo_central.root?job=dump_geometry
```
This will store this file with the name `eic.root` in this repository's directory.

Start the python webserver with the `./run` script:
```console
./run
```
Navigate to the `export_EIC.html` page at the URL indicated by the previous script, i.e. typically http://127.0.0.1:8000/export_EIC.html. This will start the conversion process and automatically start the download of the resulting `eic.gltf` file.

To hide or add which detectors are exported, modify the entries in the `hide_children` and `subparts` variables in the file `export_EIC.html`, respectively. Faster event display performance will benefit from restraint in adding geometry elements.

## Dependencies

The exporter is using several javascript libraries, most importantly [jsroot](https://github.com/root-project/jsroot) and [three.js](https://threejs.org). However, there is no need ot install them on your machine if you're happy with the default version used from the Web.

In case you want to change version, you only have to change the includes in your export script (see below)

## Using the exporter 

The exporter is essentially a script (written in javascript) which you will execute in your browser. To that purpose, you will write a small HTML page calling it by copy/pasting and adapting from the [export_LHCb.html](export_LHCb.html) example

### Structure of the export.html file

 1. Header

```html
  <head>
    <script src="https://unpkg.com/three@0.139.1/build/three.js"> </script>
    <script> import * as THREE from 'three'; </script>
    <script src="https://unpkg.com/three@0.139.1/examples/js/exporters/GLTFExporter.js"> </script>
    <script src="https://root.cern/js/latest/scripts/JSRoot.core.js"> </script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/1.3.8/FileSaver.js"></script>
  </head>
```
This is loading the various external libraries. Copy as is unless you want to use your own, local versions

 2. Data
 
 ```javascript
    import { convertGeometry } from './phoenixExport.js';
    var hide_children = [ ... ];
    var subparts = { ... };
 ```
This is the core of the conversion, where you define what to convert and how.

The first entry `hide_children` defines a list of path prefixes matching the subparts to be ignored.
Any entry in the geometry starting with one of these prefix will be dropped as well as all its children

Here is an example of a potential hide_chidren declaration :
```javascript
    var hide_children = [
        "_dd_Geometry_BeforeMagnetRegion_VP_Supports_lvDeliveryPipe",
        "_dd_Geometry_BeforeMagnetRegion_Rich1_lvRich1PhDetSupFrame"
    ];
```

See [export_LHCb.html](export_LHCb.html) for a more complete example

The `subpart` dictionnary defines the pieces of geometry you want to keep. Each entry is a set of volumes
matching one item in the phoenix "Detector" menu :
  * the key is the menu item name, including its hierarchy, with ' > ' as a separator.
    So "a > b > c" will be entry c in submenu b of menu a
  * the value is an array of 2 items :
    * a list of prefixes of paths to consider. These paths and all their children (but the ones is `hide_children`
      will be part of the current entry
    * a boolean or a float between 0 and 1 defining the initial visibility of the entry in phoenix
      * false means not visible
      * true means visible
      * float means visible with that opacity
Here is an example of a potential subpart declaration, with all 3 items within a `VP` submenu :
```javascript
    var subparts = {
        "VP > RFFoil" : [["_dd_Geometry_BeforeMagnetRegion_VP_lvVP_pvVPLeft_pvLeftRFFoil",
                          "_dd_Geometry_BeforeMagnetRegion_VP_lvVP_pvVPRight_pvRightRFFoil"], false], // not visible
        "VP > Modules" : [["_dd_Geometry_BeforeMagnetRegion_VP_lvVP_pvVPLeft_pvModule",
                           "_dd_Geometry_BeforeMagnetRegion_VP_lvVP_pvVPRight_pvModule"], true],      // visible
        "VP > Structure" : [["_dd_Geometry_BeforeMagnetRegion_VP_lvVP_pvVPLeft_pvLeftDetSup",
                             "_dd_Geometry_BeforeMagnetRegion_VP_lvVP_pvVPRight_pvRightDetSup"], .3]  // opacity 0.3
    }; 
```

See [export_LHCb.html](export_LHCb.html) for a more complete example

 3. Calling the exporter code

```javascript
    convertGeometry("./LHCb.root", "lhcb.gltf", 4, subparts, hide_children);
```
Here we call the conversion function. Parameters are :
 * the input file name
 * the name of the output file
 * the maximum depth to consider for the conversion. Anything below will be discarded
 * hide_children array of paths prefix for nodes that should be ignored (see Data above)
 * subparts definition of the subparts to create in the geometry (see Data above)


### How to run the exporter: 

Two main steps :
  * execute `run` to start a local web server on your PC serving your freshly created `export.html` file
  * point your browser to [http://localhost:8000/export.html](http://localhost:8000/export.html)
  
That's it, you should be able to download the GLFT resulting file.
You can then open it with [phoenix](https://github.com/HSF/phoenix)

