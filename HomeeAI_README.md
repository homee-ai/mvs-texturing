# HomeeAI's MVS-Texturing

This repository is a fork of the [official mvs-texturing
repository](https://github.com/nmoehrle/mvs-texturing).

---

## Prerequisites

To build and run `mvs-texturing`, ensure the following dependencies are installed:

- **CMake** >= 3.28  
- **Git**  
- **Make**  
- **GCC/G++** >= 12  
- **Libraries**: `libpng`, `libjpeg`, `libtiff` and `libtbb` (may not be required)
  ```shell
  apt update && apt install libpng-dev libjpeg-dev libtiff-dev libtbb-dev
  ```

Additional dependencies are automatically downloaded and built by the system via
`elibs/CMakeLists.txt`:

- [rayint](https://github.com/nmoehrle/rayint)
- [Eigen](http://eigen.tuxfamily.org)  
- [Multi-View Environment (MVE)](https://github.com/nmoehrle/mve.git)  
- [mapMAP](https://github.com/dthuerck/mapmap_cpu.git)  

**Note:** It may be better to update Eigen and MVE to the latest version.

## Build MVS-Texturing

Use the following command to build:

```
mkdir -p build && cd build && cmake .. && make -j; cd ..
```

**TODO:** Modularize external libraries and use modern CMake
commands.

## Prepare 3D Model and Camera Parameters

To prepare your input data, follow the preprocessing script
`scripts/mvs_texturing_preprocess_camera_parameters.py` in
the[`inpaint-3d-struct`
repository](https://github.com/homee-ai/inpaint-3d-struct).

Essentially, `mvs-texturing` only generates good results if input data has the
correct format:

- **Mesh Quality:** Ensure the input mesh has appropriate triangle sizes. Large
  triangles may result in incomplete texturing if no single image fully covers a
  triangle.

- **Camera Parameters:** The .cam file must follow this format:

    ```
    tx ty tz r00 r01 r02 r10 r11 r12 r20 r21 r22  # extrinsic
    focal_length distort_1 distort_2 pixel_aspect_ratio principal_point_x principal_point_y  # intrinsic
    ```

    Please read
    [mve/libs/mve/camera.h](https://github.com/simonfuhrmann/mve/blob/master/libs/mve/camera.h)
    for more information.


## Running MVS-Texturing

After `mvs-texturing` is built and input data is prepared, we can start texturizing meshes. The
following command

```shell
# Keep unseen faces
rm -r output \
&& mkdir -p output \
&& ./build/apps/texrecon/texrecon \
--tone_mapping=gamma \
--outlier_removal=gauss_damping \
--view_selection_model \
--keep_unseen_faces \
./input/img_cam/ \
./input/scene.ply \
./output/textured_mesh

# Assign color to unseen faces
rm -r output \
&& mkdir -p output \
&& ./build/apps/texrecon/texrecon \
--tone_mapping=gamma \
--outlier_removal=gauss_damping \
--view_selection_model \
--keep_unseen_faces \
--color_unseen_faces=192,192,192 \
./input/img_cam/ \
./input/scene.ply \
./output/textured_mesh
```

takes the images, camera parameters, and a `scene.ply` file as input and produces
the results in a destination path prefixed with `./output/textured_mesh`, as
shown below: 

```
textured_mesh.conf
textured_mesh.mtl
textured_mesh.obj
textured_mesh_data_costs.spt
textured_mesh_labeling.vec
textured_mesh_material0000_map_Kd.png
textured_mesh_material0001_map_Kd.png
textured_mesh_view_selection.mtl
textured_mesh_view_selection.obj
textured_mesh_view_selection_material0000_map_Kd.png
textured_mesh_view_selection_material0001_map_Kd.png
```

In general, it's better not to skip global and local seam leveling, as they
produce more visually appealing results with fewer seams in the mesh textures.
