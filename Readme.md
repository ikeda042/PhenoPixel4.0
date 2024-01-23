# PhenoPixel4.0

<div align="center">

![Start-up window](docs_images/Schema_new.png)

</div>
PhenoPixel4.0 is an OpenCV-based image processing program designed for automating the extraction of cell images from a large number of images (e.g., multiple nd2 files). 



<div align="center">

![Start-up window](docs_images/manual_detect_demo.gif)

</div>

It is also capable of detecting the contours of cells manually as shown so that all the phenotypic cells can be equally sampled.

This program is Python-based and utilizes Tkinter for its GUI, making it cross-platform, and has been primarily tested on Windows 11 and MacOS Sonoma 14.0.

# Installation & Setup
1. Install `python 3.8` or higher on your computer.
2. Clone this repository to your computer. (e.g., on visual studio code)
```bash
https://github.com/ikeda042/PhenoPixel4.0.git
```
3. Install the required packages with the following commmand in the root directory of the repository.
```bash
pip install -r app/requirements.txt
```

# Usage
1. Go to the root directory and run `PhenoPixel4.py`
```bash
python PhenoPixel4.py
```
After running the scripts, the landing window automatically pops up. 

![Start-up window](docs_images/landing_window.png)

2. Click "Select File" to choose a file. (file ext must be .nd2/.tif)
   

Input parameters are listed below.

| Parameters | Type | Description |
| :---: | :---: | :--- |
| Parameter 1 | int [0-255] | Lower threshold for Canny algorithm.|
| Parameter 2 | int [0-255] | Higher threshold for Canny algorithm.|
|Image Size | int | Size for square for each cell.|
|Mode| Literal | `all` for general analysis including cell extraction, `Data Analysis` for only data analysis using existing database(.db),  `Data Analysis(all db)` for sequentially read all the databases in the root directly, and `Delete All` to clear unused files.|
|Layer Mode|Literal|Dual(PH,Fluo1,Fluo2)/Single(PH)/Normal(PH,Fluo1)|

For exmaple, if you have an nd2 file structured like PH_0, Fluo_0, PH_1, Fluo_1..., `Normal` Mode works the best.

3. Click "Run" to start the program.
4. Image labeling application window pops up when done with cell extraction.
5. Choose arbitrary label for each and press "Submit" or simply press Return key. (Default value is set to N/A) You can use the label later to analyse cells. (e.g., only picking up cells with label 1)
![Start-up window](docs_images/image_labeling_app.png)
6. Close the window when reached the last cell, then database will automatically be created.

# Database
## image_labels.db

Each row has the following columns:

| Column Name | Data Type | Description                   |
|-------------|-----------|-------------------------------|
| id          | int       | Unique ID                     |
| image_id    | str       | Cell id                       |
| label       | str       | Label data manually chosen    |

## filename.db
| Column Name      | Data Type      | Description                                         |
|------------------|----------------|-----------------------------------------------------|
| id               | int            | Unique ID                                           |
| cell_id          | str            | Cell id (Frame n Cell n)                            |
| label_experiment | str \| Null    | Experimental label (e.g., Series1 exposure30min)    |
| manual_label     | str \| Null    | Label data from image_labels.db with respect to cell ID |
| perimeter        | float          | Perimeter                                           |
| area             | float          | Area                                                |
| image_ph         | BLOB           | PH image in Square block (image size x image size)  |
| image_flup1      | BLOB \| Null   | Fluo 1 image                                        |
| image_flup2      | BLOB \| Null   | Fluo 2 image                                        |
| contour          | BLOB           | 2D array cell contour                               |


# File Structure
This is the overview of the program file structure.

```bash
|-- PhenoPixel 4.0
    |-- PhenoPixel4.py
    |-- demo.py
    |-- Cell
        |-- 3dplot
        |-- GLCM
        |-- fluo1
        |-- fluo1_incide_cell_only
        |-- fluo2
        |-- fluo2_incide_cell_only
        |-- gradient_magnitude_replot
        |-- gradient_magnitudes
        |-- histo
        |-- histo_cumulative
        |-- peak_path
        |-- ph
        |-- projected_points
        |-- replot
        |-- replot_map
        |-- sum_brightness
        |-- unified_cells
    |-- RealTimeData
        |-- 3dplot.png
        |-- fluo1.png
        |-- fluo1_incide_cell_only.png
        |-- histo_cumulative_delta.png
        |-- peak_path.png
        |-- ph.png
        |-- re_replot.png
        |-- replot.png
        |-- replot_grad_magnitude.png
        |-- sum_brightness.png
    |-- app
        |-- .gitignore
        |-- main.py
        |-- nd2extract.py
        |-- requirements.txt
        |-- test_database.db
        |-- Cell
            |-- 3dplot
            |-- GLCM
            |-- fluo1
            |-- fluo1_incide_cell_only
            |-- fluo2
            |-- fluo2_incide_cell_only
            |-- gradient_magnitudes
            |-- histo
            |-- histo_cumulative
            |-- peak_path
            |-- ph
            |-- projected_points
            |-- replot
            |-- replot_map
            |-- sum_brightness
            |-- unified_cells
        |-- DataAnalysis
            |-- .gitignore
            |-- SVD.py
            |-- calc_cell_length.py
            |-- combine_images.py
            |-- components.py
            |-- cumulative_plot_analysis.py
            |-- data_analysis.py
            |-- data_analysis_light.py
            |-- fluo_localization_heatmap_analysis.py
            |-- old_schema_patch.py
            |-- peak_paths_plot.py
            |-- skewness_analysis_for_periplasm.py
            |-- utils
                |-- .gitignore
                |-- CDF_analysis.py
        |-- pyfiles
            |-- .gitignore
            |-- app.py
            |-- calc_center.py
            |-- crop_contours.py
            |-- database.py
            |-- delete_all.py
            |-- extract_tiff.py
            |-- image_process.py
            |-- initialize.py
            |-- unify_images.py
```

- `PhenoPixel4.py`: Provides GUI and file selection features using tkinter.
- `main.py`: Central functionalities including image processing and data analysis.
- `nd2extract.py`: Data extraction from ND2 files.
- `app.py`: GUI part of the application using tkinter and SQLite.
- `calc_center.py`: Calculates the center of contours in images using OpenCV.
- `crop_contours.py`: Processes images to crop contours.
- `extract_tiff.py`: Extraction and processing of TIFF files.
- `image_process.py`: Integrates various custom modules for image processing.
- `initialize.py`: Initial setup for image processing.
- `unify_images.py`: Combines multiple images into a single output.
- `demo.py`: Data analysis demonstration using `test_database.db`
  

# Output Files/Folders
These folders are automatically created once the scripts start.
## TempData/
**app_data**

All detected cells are labeled with a Cell ID (e.g., F1C4) and stored in this folder. The cells are in the square of "Image Size". Note that invalid cells (e.g., misditected cells) are also stored here.

**Fluo1**

The entire image of each frame for Fluo1 is included.

**Fluo2**

The entire image of each frame for Fluo2 is included.

**PH**

The entire image of each frame for PH is included.

## ph_contours/
This folder contains the entire images of each PH frame with detected contours (in green) on the cells.

# Algorithms

## Cell Elongation Direction Determination Algorithm

### Objective:
To implement an algorithm for calculating the direction of cell elongation.

### Method: 

In this section, we consider the elongation direction determination algorithm with regard to the cell with contour shown in Fig.1 below. 

Scale bar is 20% of image size (200x200 pixel, 0.0625 µm/pixel)

<div align="center">

![Start-up window](docs_images/algo1.png)  

</div>

<p align="center">
Fig.1  <i>E.coli</i> cell with its contour (PH Left, Fluo-GFP Center, Fluo-mCherry Right)
</p>

Consider each contour coordinate as a set of vectors in a two-dimensional space:

$$\mathbf{X} = 
\left(\begin{matrix}
x_1&\cdots&x_n \\
y_1&\cdots&y_n 
\end{matrix}\right)^\mathrm{T}\in \mathbb{R}^{n\times 2}$$

The covariance matrix for $\mathbf{X}$ is:

$$\Sigma =
 \begin{pmatrix} V[\mathbf{X_1}]&Cov[\mathbf{X_1},\mathbf{X_2}]
 \\ 
 Cov[\mathbf{X_1},\mathbf{X_2}]& V[\mathbf{X_2}] \end{pmatrix}$$

where $\mathbf{X_1} = (x_1\:\cdots x_n)$, $\mathbf{X_2} = (y_1\:\cdots y_n)$.

Let's define a projection matrix for linear transformation $\mathbb{R}^2 \to \mathbb{R}$  as:

$$\mathbf{w} = \begin{pmatrix}w_1&w_2\end{pmatrix}^\mathrm{T}$$

Now the variance of the projected points to $\mathbb{R}$ is written as:
$$s^2 = \mathbf{w}^\mathrm{T}\Sigma \mathbf{w}$$

Assume that maximizing this variance corresponds to the cell's major axis, i.e., the direction of elongation, we consider the maximization problem of the above equation.

To prevent divergence of variance, the norm of the projection matrix is fixed at 1. Thus, solve the following constrained maximization problem to find the projection axis:

$$arg \max (\mathbf{w}^\mathrm{T}\Sigma \mathbf{w}), \|\mathbf{w}\| = 1$$

To solve this maximization problem under the given constraints, we employ the method of Lagrange multipliers. This technique introduces an auxiliary function, known as the Lagrange function, to find the extrema of a function subject to constraints. Below is the formulation of the Lagrange multipliers method as applied to the problem:

$$\cal{L}(\mathbf{w},\lambda) = \mathbf{w}^\mathrm{T}\Sigma \mathbf{w} - \lambda(\mathbf{w}^\mathrm{T}\mathbf{w}-1)$$

At maximum variance:
$$\frac{\partial\cal{L}}{\partial{\mathbf{w}}} = 2\Sigma\mathbf{w}-2\lambda\mathbf{w} = 0$$

Hence, 

$$ \Sigma\mathbf{w}=\lambda\mathbf{w} $$

Select the eigenvector corresponding to the eigenvalue where λ1 > λ2 as the direction of cell elongation. (Longer axis)

### Result:

Figure 2 shows the raw image of an <i>E.coli </i> cell and the long axis calculated with the algorithm.


<div align="center">

![Start-up window](docs_images/algo1_result.png)  

</div>

<p align="center">
Fig.2  <i>E.coli</i> cell with its contour (PH Left, Replotted contour with the long axis Right)
</p>


## Basis conversion Algorithm

### Objective:

To implement an algorithm for replacing the basis of 2-dimentional space of the cell with the basis of the eigenspace(2-dimentional).

### Method:


Let 

$$ \mathbf{Q}  = \begin{pmatrix}
    v_1&v_2
\end{pmatrix}\in \mathbb{R}^{2\times 2}$$

$$\mathbf{\Lambda} = \begin{pmatrix}
    \lambda_1& 0 \\
    0&\lambda_2
\end{pmatrix}
(\lambda_1 > \lambda_2)$$

, then the spectral factorization of Cov matrix of the contour coordinates can be writtern as:

$$\Sigma =
 \begin{pmatrix} V[\mathbf{X_1}]&Cov[\mathbf{X_1},\mathbf{X_2}]
 \\ 
 Cov[\mathbf{X_1},\mathbf{X_2}]& V[\mathbf{X_2}] \end{pmatrix} = \mathbf{Q}\mathbf{\Lambda}\mathbf{Q}^\mathrm{T}$$

Hence, arbitrary coordinates in the new basis of the eigenbectors can be written as:

$$\begin{pmatrix}
    u_1&u_2
\end{pmatrix}^\mathrm{T} = \mathbf{Q}\begin{pmatrix}
    x_1&y_1
\end{pmatrix}^\mathrm{T}$$

### Result:

Figure 3 shows contour in the new basis 

$$\begin{pmatrix}
    u_1&u_2
\end{pmatrix}$$ 

<div align="center">

![Start-up window](app/images_readme/base_conv.png)  
</div>
<p align="center">
Fig.3  Each coordinate of contour in the new basis (Right). 
</p>


## Cell length calculation Algorithm

### Objective:

To implement an algorithm for calculating the cell length with respect to the center axis of the cell.

### Method:

<i>E.coli</i> expresses filamentous phenotype when exposed to certain chemicals. (e.g. Ciprofloxacin)

Figure 4 shows an example of a filamentous cell with Ciprofloxacin exposure. 

<div align="center">

![Start-up window](docs_images/fig4.png)  

</div>

