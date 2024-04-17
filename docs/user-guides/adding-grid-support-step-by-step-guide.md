# Adding Support for New Grids - Step-by-Step Guide

The purpose of this guide is to outline all the necessary steps for running E3SM on a new grid for the atmosphere and land components. The process is similar for uniform and regionally refined grids, although regionally refined cases will likely require some special considerations which will be noted where appropriate.

If you wish to add a new ocean and sea-ice mesh you will need to use the compass tool to generate the mesh and dynamically adjusted initial condition. This procedure is detailed in a separate tutorial:
<https://mpas-dev.github.io/compass/latest/tutorials/dev_add_rrm.html>

<!-- disable certain linter checks here to allow vertical alignment of links -->
<!-- markdownlint-disable MD039 --> <!-- no-space-in-links -->
<!-- markdownlint-disable MD042 --> <!-- no-empty-links -->
1. [Generate a new grid file                                  ](adding-grid-support-step-by-step-guide/generate-new-grid-file.md)
1. [Generate mapping files                                    ]()
1. [Generate domain files                                     ]()
1. [Generate a topography file                                ]()
1. [Generate an initial condition for the atmosphere          ]()
1. [Generate land surface input data (*fsurdat*)              ]()
1. [Generate an initial condition for land (*finidat*)        ]()
1. [Create a new dry deposition file (*depends on use case*)  ]()
1. [Modify E3SM configuration files to support the new grid   ]()
