# OpenFOAM_Python_Training_Material
Training material and necessary files for First UK-India OpenFOAM Symposium

# Getting Started

## Accessing Files 

The workshop training material is available on Github, and can be downloaded using the following command:

```console
git clone https://github.com/goswami-13/OpenFOAM_Python_Training_Material.git
```

**Tip:** Clone/download this repository and place it in your working directory for ease!!! 

## Setting up Python

At this point, I assume you have the basic understanding of Python programming and installing in python. Follow the steps below for installing the required dependencies for this workshop.

1. Navigate to where you downloaded the training files and look for `requirements.txt' file.
2. Install the dependencies noted in this file using `pip install` or `python -m`. Both methods are shown below:

   Using `pip` :
   ```console
   pip install -r requirements.txt
   ```

   Using `python -m` :
   ```console
   python -m pip install -r requirements.txt
   ```
3. Wait for the installation to finish

Your Python environment is set!!!

In case you are new to Python, fear not!! [Here](https://www.codingforentrepreneurs.com/guides/install-python-on-windows/) is a handy guide for installing and setting up python on windows.

# Workshop Deliverables

1. Extracting and plotting velocity and pressure along the probe points.

2. Extracting and plotting force coefficient data.

3. Computing FFT using probe point pressure and force data.

4. Extracting and plotting Data along multiple lines from the simulations.

5. Extracting 2D slice data and visualizing it.

6. Some hints for advance analysis and post-processing such as POD, DMD, and animations!!!

# Jupyter File

I prefer using Jypyter notebook (In VSCode) as by basic python working environment. As such this workshop comes with a handy file named `Workshop.ipynb`. The above deliverables and their codes are given in this file.

# OpenFOAM Cases

We will be using two OpenFOAM case studies as an example for this workshop.

1. Flow over 2D square cylinder at $Re = 100$ (Bai & Alam 2018).
2. Two side-by-side cylinders with gap ratio of $4$ at $Re = 75$ (Ma et al. 2017).

Both case studies are incompressible, 2D, Transient and unsteady simulations run in OpenFOAM using `pimpleFoam`. Both cases are run until a fully-developed state is achieved.

## Necessary code snippets

### Extracting Data along multiple lines from the simulations.

Step 1 : create a file names Multiple lines in system folder of your case : `touch MultipleLines`

Step 2 : Copy the following function object into the file

```bash
/*--------------------------------*- C++ -*----------------------------------*\
  =========                 |
  \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox
   \\    /   O peration     | Website:  https://openfoam.org
    \\  /    A nd           | Version:  7
     \\/     M anipulation  |
-------------------------------------------------------------------------------
Description
    Writes graph data for specified fields along a line, specified by start
    and end points.

\*---------------------------------------------------------------------------*/


// Sampling and I/O settings
#includeEtc "caseDicts/postProcessing/graphs/sampleDict.cfg"

type            sets;
libs            ("libsampling.so");

writeControl    writeTime;

interpolationScheme cellPoint;

setFormat   csv;

setConfig
{
    type    uniform;
    axis    xyz;  // x, y, z, xyz
	nPoints	1000;
}

sets
(
    line1
    {
        $setConfig;
        start   (0 0 0.05);
		end     (2.35 0 0.05);
    }
	
	line2
    {
        $setConfig;
        start (0.4 1.2 0.05);
        end   (0.4 -1.2 0.05);
    }
	
	line3
    {
        $setConfig;
        start (0.8 1.2 0.05);
        end   (0.8 -1.2 0.05);
    }
);

fields  (U p UMean pMean UPrime2Mean);

// ************************************************************************* //
```

Step 3 : Use CLI post-processing utility to extract data along the line : `postProcess -func MultipleLines`

Step 4 : Read `csv` files into python

### Extracting 2D slice data from simulations.

Step 1 : Compute vorticity using OpenFOAM CLI : `postProcess -func vorticity`

Step 2 : Create a file names `surfaces` in system folder of your case

Step 3 : Copy the following function object into the file

```bash
/*--------------------------------*- C++ -*----------------------------------*\
  =========                 |
  \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox
   \\    /   O peration     |
    \\  /    A nd           | Web:      www.OpenFOAM.com
     \\/     M anipulation  |
-------------------------------------------------------------------------------
Description
    Writes out values of fields from cells nearest to specified locations.

\*---------------------------------------------------------------------------*/
surfaces
{
    type            surfaces;
    libs            ("libsampling.so");
    writeControl   	timeStep;
    writeInterval   142;

    surfaceFormat   vtk;
    fields          (p U vorticity);

    interpolationScheme cellPoint;

    surfaces
    {
        zNormal
        {
            type        cuttingPlane;
            point       (0 0 0.05);
            normal      (0 0 1);
            interpolate true;
        }
    };
};
// ************************************************************************* //
```

Step 4 : Extract the 2D surface slice using OpenFOAM CLI : `postProcess -func surfaces`

Step 5 : Read the surface VTK into python and visualize!!!


