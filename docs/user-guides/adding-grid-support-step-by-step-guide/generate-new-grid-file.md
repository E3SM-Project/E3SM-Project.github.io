# Generate a new Grid File

In order to generate mapping files between a new atmosphere grid and the surface component grids, we need a file that describes the new grid. [TempestRemap](https://github.com/ClimateGlobalChange/tempestremap) is our preferred tool for grid file generation because it can handle the spectral element grids used by the atmosphere dycore. The initial grid file will be saved in an "exodus" file with a `.g` extension (see [Types of Grid Description Files](../adding-grid-support-grid-types.md) for more info). TempestRemap can be installed via conda.

## Generating a Standard Exodus Grid File

Once TempestRemap is in our environment we can easily generate an exodus file by calling TempestRemap directly:

```bash
GenerateCSMesh --alt --res 4 --file ne4.g
```

## Generating a Regionally Refined Grid File

For a regionally refined mesh (RRM) [SQuadGen](https://github.com/ClimateGlobalChange/squadgen) is used to define the refined area(s). [This tutorial](generate-RRM-grid-file.md) includes details and examples of using SQuadGen to generate RRM grid files.

The naming convention for RRM grid files should follow:

```bash
<refined_area_name>_<base_resolution>x<refinement_level>.g
```

For example, for a RRM with 4x refinement from ne30 to ne120 over CONUS, we should use the convention conus_ne30x4.g (note that existing meshes may have used the old naming convention `<area of refinement>x<refinement level>v<version>.g`, but future meshes should use the new naming convention).

The Exodus file contains only information about the position of the spectral element on the sphere. For SE aware utilities such as TempestRemap, they can use the polynomial order and the reference element map to fill in necessary data such as the locations of the nodal GLL points. For non-SE aware utilities, we need additional meta data, described in the next section.  
