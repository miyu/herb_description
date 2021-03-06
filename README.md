# herb_description

herb_description contains URDF and SRDF descriptions of HERB, a bimanual mobile
manipulation robot designed and built by the
[Personal Robotics Lab](https://personalrobotics.ri.cmu.edu) in the
[Robotics Institute](https://www.ri.cmu.edu) at
[Carnegie Mellon University](http://www.cmu.edu). The URDF descriptions follow [REP 120](http://www.ros.org/reps/rep-0120.html).


## SolidWorks Export

This package contains several URDF files that are directly exported from
[SolidWorks](http://www.solidworks.com) using the
[SW2URDF](http://wiki.ros.org/sw_urdf_exporter). These files are:

- `BHD280_URDF.URDF`:
  [BarrettHand](http://www.barrett.com/robot/products-hand.htm) model (the
  BH-280 hardware revision)
- `HERB_BASE_URDF.URDF`: HERB torso and head
- `WAM_URDF.URDF`: [Barrett WAM](http://www.barrett.com/robot/products-arm.htm)
  model (7-DOF, with wrist and force/torque sensor)

These files **should not** be
edited by hand since they will be overwritten by future SolidWorks exports.


## URDF Transformation

The URDF exported from SolidWorks does not correctly specify mass, inertia
properties, and joint limits. These properties are patched into the raw
URDF exported by SolidWorks using the `postprocess_urdf.py` script.

Specifically, the following transformations occur at build time:

- `BHD280_URDF.URDF` + `config/bh280_params.urdf.xacro` -> `bh280_raw.urdf`
- `HERB_BASE_URDF.URDF` + `config/herb_params.urdf.xacro` -> `herb_base_raw.urdf`
- `WAM_URDF.URDF` + `config/wam_params.urdf.xacro` -> `wam_raw.urdf`

These output files, with the suffix `_raw.urdf`, are an intermediate build
artifact and **should not** be used directly. See below if you you want to
include a BH-280 or WAM model as part of a larger URDF file.


## xacro Transformation 

The files generated above are wrapped in thin
[xacro](http://wiki.ros.org/xacro) wrappers using the `postprocess_xacro.py`
script. This script performs several transformations of the input file:

1. Wrap the entire URDF file in a `xacro:macro` tag with a `prefix` argument.
2. Prepend `prefix` onto all link and joint names
3. Repair any incorrect `package://` URIs generated by `SW2URDF` (optional)
4. Assign a default color to all links (optional)
5. Replace the collision geometry with primitives and/or meshes (optional)
6. Replace visual STL files with COLLADA models (optional)

Together, transformations (1) and (2) allow one URDF model exported by
SolidWorks to be instantiated more than once in a larger xacro file. This is
critical since HERB has two Barrett WAMs and BarrettHand end-effectors. The
following transformations occur at build time:

- `bh280_raw.urdf` ->  `bh280.urdf.xacro`
- `herb_base_raw.urdf` -> `herb_base.urdf.xacro`
- `wam_raw.urdf` ->  `wam.urdf.xacro`.

These output files are placed in the `devel` space in your Catkin workspace and
are intended for re-use. You **should** use these files when constructing
larger models out of these components.


## xacro Expansion

Finally, the `herb.urdf.xacro` file is expanded into `herb.urdf` by directly
invoking xacro. This xacro file specifies the overall HERB description by
instantiating the `bh280.urdf.xacro`, `herb_base.urdf.xacro`, and
`wam.urdf.xacro` xacro macros generated in the previous build step. Similarly,
a `herb.srdf` SRDF file is generated from the `herb.srdf.xacro` file **You
should only include the `herb.urdf` and `herb.srdf` files** generated in this
build step for most applications.

As a convenience, the following two URDF models are also generated:

- `bh280_standalone.urdf` - standalone BarrettHand 
- `wam_standalone.urdf` - 7-DOF WAM with force/torque sensor and wrist
- `wam_bh280_standalone.urdf` - model consisting of `bh280_standalone.urdf`
  attached to the wrist of `wam_standalone.urdf`


## Notes on Visual Geometry

Getting the COLLADA (i.e. `.dae`) files to both display properly in RViz
andload in OpenRAVE takes some effort. Specifically:

- When generating new meshes, make sure the correct schema (`1.4`, not `1.5`) is
  used. The second line of the file should read:
```xml
<COLLADA xmlns="http://www.collada.org/2005/11/COLLADASchema" version="1.4.0">
```
- Depending on the software that is exporting the mesh, you must take care to
  ensure that the resulting file has the `<up_axis>Y_UP</up_axis>` tag
- All units should be in meters


## LICENSE

Unless otherwise noted, herb_description is licensed under a BSD license (see
`LICENSE.bsd`). The COLLADA visual geometry for the Barrett WAM and BarrettHand
are licensed under the GPL (see `LICENSE.gpl`). This includes the following
files in the `meshes` directory:

- all `.dae` models except for: `head_wam1.dae`, `head_wam2.dae,
  `herb_base.dae`, `segway_wheel_left.dae`, and `segway_wheel_right.dae`
- all `.jpg` textures except for: `head_wam1.jpg`, `head_wam2.jpg`,
  `herb_base.jpg`, and `wheel.jpg`


## Contributors

PrPy is developed by the [Personal Robotics
Lab](https://personalrobotics.ri.cmu.edu) in the [Robotics
Institute](https://www.ri.cmu.edu) at [Carnegie Mellon
University](http://www.cmu.edu).

This is a non-exhaustive list of contributors:

- The URDF and SRDF packaging of HERB was done by [Michael
  Koval](https://github.com/mkoval)
- This model is based on an earlier model built by [Mike Vande
  Weghe](https://github.com/vandeweg)
- HERB's torso and head were designed, modeled, and exported by [Michael
  Dawson-Haggerty](https://github.com/mikedh)
- The visual geometry for HERB's torso and head was modeled by [Aaron
  Walsman](https://github.com/aaronwalsman).
- The Barrett WAM and BarrettHand URDF files were exported by Michael
  Dawson-Haggerty from the SolidWorks models provided by
  [Barrett Technology, Inc.](http://www.barrett.com/)
- The visual geometry for the Barrett WAM and BarrettHand is copied from models
  produced by [RE2 Inc.](http://www.resquared.com) as part of the DARPA
  Autonomous Robotic Manipulation program. This package was available under the
  GPL license (see above for more information).
- Simplified collision geometry for the entire model is courtesy of Aaron
  Walsman.

