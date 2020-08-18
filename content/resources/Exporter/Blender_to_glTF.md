# Blender to BJS, using glTF

glTF exporter will allow you to export your scene using PBR workflow.

## Features

Since Blender 2.8, glTF addon comes with Blender enabled by default. You can update it from the official [Github repo](https://github.com/KhronosGroup/glTF-Blender-IO). Official documentation is on [Blender Manual](https://docs.blender.org/manual/en/latest/addons/io_scene_gltf2.html).

It should be compatible with Blender 2.79b, but you may also note that the old exporter is  [still available](https://github.com/KhronosGroup/glTF-Blender-Exporter) ([old documentation](https://github.com/KhronosGroup/glTF-Blender-Exporter/blob/master/docs/user.md)).

Axys conventions aren't the same between Blender, BabylonJS (left handed) & glTF (right handed), so you can see below a conversion table to help you about coordinates.

To help transforming, note that the BabylonJS loader will automatically set glTF assets as children of an object:
- named `__root__`
- rotated by default to 180° on Y axys
- scaled on Z by -1


| Blender asset position | BabylonJS asset absolutePosition |
| :---: | :---: |
| X | -X |
| Y | Z |
| Z | -Y |



##  Try it out!

Once your scene is exported, you have multiple solutions to test it:

- quick check using the [sandbox](http://sandbox.babylonjs.com/)
- use the [viewer](//doc.babylonjs.com/extensions/the_babylon_viewer)
- script your own app using the [loader](/How_To/Load_From_Any_File_Type)

### Example

Let's say you have exported [WaterBottle.glb](https://github.com/KhronosGroup/glTF-Sample-Models/tree/master/2.0/WaterBottle/glTF-Binary):

- export to .gltf or .glb in a folder
- create a file named *index.html*, and copy the code above:

```html
<!doctype html>
<html>

<head>
    <title>Default .gltf loading scene</title>
    <meta charset="UTF-8">
    <!-- this link to the preview online version of BJS -->
    <script src="https://preview.babylonjs.com/babylon.js"></script>
    <!-- this is needed for BJS to load scene files -->
    <script src="https://preview.babylonjs.com/loaders/babylonjs.loaders.js"></script>
    <style>
        html,
        body {
            overflow: hidden;
            width: 100%;
            height: 100%;
            margin: 0;
            padding: 0;
        }

        #canvas {
            width: 100%;
            height: 100%;
            touch-action: none;
        }
    </style>
</head>

<body>
    <canvas id="canvas"></canvas>
    <script type="text/javascript">
        var canvas = document.getElementById("canvas");
        var engine = new BABYLON.Engine(canvas, true);

        // here the doc for Load function: //doc.babylonjs.com/api/classes/babylon.sceneloader#load
        BABYLON.SceneLoader.Load("", "WaterBottle.glb", engine, function (scene) {

            scene.createDefaultCamera(true, true, true);
            scene.createDefaultEnvironment({
                createGround: false,
                createSkybox: false
            });

            engine.runRenderLoop(function () {
                scene.render();
            });

            window.addEventListener("resize", function () {
                engine.resize();
            });
        });
    </script>
</body>

</html>

```

- double-click on the *index.html* file
  - some browsers may not want loading the scene, for some security issues (e.g.: Chrome). In this case, you have to open the html file through a webserver ([local](/resources/running_a_local_webserver_for_babylonjs) or not), or try into another browser (e.g.: Firefox, Edge)

- ... profit!

![blender gltf scene loaded in BJS](/img/exporters/blender/gltf/gltf-loaded.png)