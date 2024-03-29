# PhenoPixel4.0

<div align="center">

![Start-up window](app/images_readme/schema.png)

</div>

PhenoPixel4.0 is an OpenCV-based image processing program designed for automating the extraction of cell images from a large number of images (e.g., multiple nd2 files). 

It is also capable of detecting the contours of cells manually as shown so that all the phenotypic cells can be equally sampled.
<div align="center">

![Start-up window](app/images_readme/manual_detect_demo.gif)

</div>
This program is Python-based and utilizes Tkinter for its GUI, making it cross-platform. 

It has been primarily tested on Windows 10 and MacOS Sonoma 14.0.

# Installation 
1. Install python3.8 or higher on your computer.
2. Clone this repository to your computer. (e.g., on visual studio code)
```bash
https://github.com/ikeda042/PhenoPixel4.0.git
```
1. Install the required packages with the following commmand in the root directory of the repository.
```bash
pip install -r requirements.txt
```

# Usage
1. Go to the directory and run cytomatic4.py

```bash
cd PhenoPixel4.0
python3 PhenoPixel4.py
```

![Start-up window](app/images_readme/cytomatic4.png)

2. Click "Select File" to choose a file. (file ext must be .nd2/.tif)
3. Input parameters. 
    * Parameter 1 : int [0-255] ： Lower threshold for Canny algorithm.
    * Parameter 2 : int [Parameter 1-255] ： Higher threshold  for Canny algorithm.
    * Image Size : int ： Size for square for each cell.
    * Mode ： "all" for general analysis including cell extraction, "Data Analysis" for only data analysis using existing database(.db), "Delete All" to clear unused files.
    * Layer Mode -> Dual (PH,Fluo1,Fluo2), Single(PH), Normal(PH,Fluo1)
4. Click "Run" to start the program.
5. Image labeling application window pops up when done with cell extraction.
6. Choose arbitrary label for each and press "Submit" or simply press Return key. (Default value is set to N/A)
![Start-up window](app/images_readme/2.png)
1. Close the window when reached the last cell, then database will automatically be created.

# Database
## image_labels.db
Each cell ID has its manual_label from the application input.
![db column for image_labels.db](app/images_readme/3.png)
## filename.db
![db column for image_labels.db](app/images_readme/4.png)
Each row (representing a cell) has the following values:
* id : int : unique ID
* cell_id : str : cell id (Frame n Cell n)
* label_experiment : str | Null : experimental label (e.g. Series1 exposure30min)
* manual_label : str | Null : label data from image_labels.db with respect to cell ID 
* perimeter : float : perimeter 
* area : float : area 
* image_ph: BLOB : PH image in Square block (image size x image size)
* image_flup1 : BLOB | Null : Fluo 1 image
* image_flup2 : BLOB | Null : Fluo 2 image
* contour: BLOB : 2d array cell contour

# File Structure

- `cytomatic4.py`: Provides GUI and file selection features using tkinter.
- `main.py`: Central functionalities including image processing and data analysis.
- `nd2extract.py`: Data extraction from ND2 files.
- `app.py`: GUI part of the application using tkinter and SQLite.
- `calc_center.py`: Calculates the center of contours in images using OpenCV.
- `crop_contours.py`: Processes images to crop contours.
- `extract_tiff.py`: Extraction and processing of TIFF files.
- `image_process.py`: Integrates various custom modules for image processing.
- `initialize.py`: Initial setup for image processing.
- `unify_images.py`: Combines multiple images into a single output.

# Output Files/Folders
These folders are automatically created once the scripts start.
## ./TempData
**/app_data**

All detected cells are labeled with a Cell ID (e.g., F1C4) and stored in this folder. The cells are in the square of "Image Size". Note that invalid cells (e.g., misditected cells) are also stored here.

**Fluo1**

The entire image of each frame for Fluo1 is included.

**Fluo2**

The entire image of each frame for Fluo2 is included.

**PH**

The entire image of each frame for PH is included.

## ./ph_contours
This folder contains the entire images of each PH frame with detected contours (in green) on the cells.

# Algorithms
## Cell Elongation Direction Determination Algorithm
### Objective:

To implement an algorithm for calculating the direction of cell elongation.

### Method: 

In this section, we consider the elongation direction determination algorithm with regard to the cell with contour shown in Fig.1 below. 

Scale bar is 20% of image size (200x200 pixel, 0.0625 µm/pixel)

<div align="center">

![Start-up window](app/images_readme/5.png)  

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

Select the eigenvector corresponding to the eigenvalue where λ1 > λ2 as the direction of cell elongation.


### Result:

Figure 2 shows the raw image of an <i>E.coli </i> cell and the long axis calculated with the algorithm.

<div align="center">

![Start-up window](app/images_readme/result_1.png)  

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

![Start-up window](app/images_readme/fig4.png)  
</div>
<p align="center">
Fig.4 A filamentous <i>E.coli</i> cell (PH Left, Fluo-GFP Center, Fluo-mCherry Right).
</p>

Thus, the center axis of the cell, not necessarily straight, is required to calculate the cell length. 

Using the aforementioned basis conversion algorithm, first we converted the basis of the cell contour to its Cov matrix's eigenvectors' basis.

Figure 5 shows the coordinates of the contour in the new basis. 

<div align="center">

![Start-up window](app/images_readme/fig5.png)  
</div>
<p align="center">
Fig.5 The coordinates of the contour in the new basis (PH Left, contour in the new basis Right).
</p>

We then applied least aquare method to the coordinates of the contour in the new basis.

Let the contour in the new basis

$$\mathbf{C} = \begin{pmatrix}
    u_{1_1} &\cdots&\ u_{1_n} \\ 
    u_{2_1} &\cdots&\ u_{2_n} 
\end{pmatrix} \in \mathbb{R}^{2\times n}$$

then regression with arbitrary k-th degree polynomial (i.e. the center axis of the cell) can be expressed as:
$$f\hat{(u_1)} = \theta^\mathrm{T} \mathbf{U}$$

where 

$$\theta = \begin{pmatrix}
    \theta_k&\cdots&\theta_0
\end{pmatrix}^\mathrm{T}\in \mathbb{R}^{k+1}$$

$$\mathbf{U} = \begin{pmatrix}
    u_1^k&\cdots u_1^0
\end{pmatrix}^\mathrm{T}$$

the parameters in theta can be determined by normal equation:

$$\theta = (\mathbf{W}^\mathrm{T}\mathbf{W})^{-1}\mathbf{W}^\mathrm{T}\mathbf{f}$$

where

$$\mathbf{W} = \begin{pmatrix}
    u_{1_1}^k&\cdots&1 \\
     \vdots&\vdots&\vdots \\
     u_{1_n}^k&\cdots&1 
\end{pmatrix} \in \mathbb{R}^{n\times k +1}$$

$$\mathbf{f} = \begin{pmatrix}
    u_{2_1}&\cdots&u_{2_n}
\end{pmatrix}^\mathrm{T}$$

Hence, we have obtained the parameters in theta for the center axis of the cell in the new basis. (fig. 6)

Now using the axis, the arc length can be calculated as:

$$\mathbf{L} = \int_{u_{1_1}}^{u_{1_2}} \sqrt{1 + (\frac{d}{du_1}\theta^\mathrm{T}\mathbf{U})^2} du_1 $$

**The length is preserved in both bases.**

We rewrite the basis conversion process as:

$$\mathbf{U} = \mathbf{Q}^\mathbf{T} \mathbf{X}$$

The inner product of any vectors in the new basis $\in \mathbb{R}^2$ is 

$$ \|\mathbf{U}\|^2 = \mathbf{U}^\mathrm{T}\mathbf{U} = (\mathbf{Q}^\mathrm{T}\mathbf{X})^\mathrm{T}\mathbf{Q}^\mathbf{T}\mathbf{X} = \mathbf{X}^\mathrm{T}\mathbf{Q}\mathbf{Q}^\mathrm{T}\mathbf{X} \in \mathbb{R}$$

Since $\mathbf{Q}$ is an orthogonal matrix, 

$$\mathbf{Q}^\mathrm{T}\mathbf{Q} = \mathbf{Q}\mathbf{Q}^\mathrm{T} = \mathbf{I}$$

Thus, 

$$\|\mathbf{U}\|^2 = \|\mathbf{X}\|^2$$

Hence <u>the length is preserved in both bases.</u> 


### Result:

Figure 6 shows the center axis of the cell in the new basis (4-th polynominal).

<div align="center">

![Start-up window](app/images_readme/fig6.png)  
</div>
<p align="center">
Fig.6 The center axis of the contour in the new basis (PH Left, contour in the new basis with the center axis Right).
</p>

### Choosing the Appropriate K-Value for Polynomial Regression

By default, the K-value is set to 4 in polynomial regression. However, this may not be sufficient for accurately modeling "wriggling" cells.

For example, Figure 6-1 depicts a cell exhibiting extreme filamentous changes after exposure to Ciprofloxacin. The center axis as modeled does not adequately represent the cell's structure.

<div align="center">

![Start-up window](app/images_readme/choosing_k_1.png)  
</div>
<p align="center">
Fig.6-1  An extremely filamentous cell. (PH Left, contour in the new basis with the center axis Right).
</p>

The center axis (in red) with K = 4 does not fit as well as expected, indicating a need to explore higher K-values (i.e., K > 4) for better modeling.

Figure 6-2 demonstrates fit curves (the center axis) for K-values ranging from 5 to 10.

<div align="center">

![Start-up window](app/images_readme/result_kth10.png)  
</div>
<p align="center">
Fig.6-2: Fit curves for the center axis with varying K-values (5 to 10).
</p>

As shown in Fig. 6-2, K = 8 appears to be the optimal value. 

However, it's important to note that the differences in calculated arc lengths across various K-values fall within the subpixel range.

Consequently, choosing K = 4 might remain a viable compromise in any case.

## Quantification of Localization of Fluorescence
### Objective:

To accurately quantify the localization of fluorescence within cells.

### Method:

Quantifying the localization of fluorescence is straightforward in cells with a "straight" morphology(fig. 7-1). 

<div align="center">

![Start-up window](app/images_readme/fig_straight_cell.png)  
</div>
<p align="center">
Fig.7-1: An image of an <i>E.coli</i> cell with a straight morphology.
</p>

However, challenges arise with "curved" cells(fig. 7-2).

To address this, we capitalize on our pre-established equation representing the cellular curve (specifically, a quadratic function). 

This equation allows for the precise calculation of the distance between the curve and individual pixels, which is crucial for our quantification approach.

The process begins by calculating the distance between the cellular curve and each pixel. 

This is achieved using the following formula:

An arbitrary point on the curve is described as:
$$(u_1,\theta^\mathrm{T}\mathbf{U}) $$
The minimal distance between this curve and each pixel, denoted as 
$(p_i,q_i)$, is calculated using the distance formula:

$$D_i(u_1) = \sqrt{(u_1-p_i)^2+(f\hat{(u_1)} - q_i)^2}$$

Minimizing $D_i$ with respect to $u_1$ ensures orthogonality between the curve and the line segment joining $(u_1,\theta^\mathrm{T}\mathbf{U})$ and $(p_i,q_i)$ 

This orthogonality condition is satisfied when the derivative of $D_i$ with respect to $u_1$ is zero.

The optimal value of $u_1$, denoted as $u_{1_i}^\star$, is obtained by solving 

$$\frac{d}{du_1}D_i = 0\:\forall i$$

for each pixel  $(p_i,q_i)$. 

Define the set of solution vectors as 
$$\mathbf{U}^\star = \lbrace (u_{1_i}^\star,f\hat{(u_{1_i}^\star)})^\mathrm{T} : u_{1_i}^\star \in u_1 \rbrace \in \mathbb{R}^{2\times n}$$

, where $f\hat{(u_{1_i}^\star)}$ denotes the correspoinding function value.


It should be noted that the vectors in $\mathbf{U}^\star$ can be interpreted as the projections of the pixels $(p_i,q_i)$ onto the curve.

Define the set of projected vectors $\mathbf{P}^\star$ such that each vector in this set consists of the optimal parameter value $u_{1_i}^\star$ and the corresponding fluorescence intensity, denoted by $G(p_i,q_i)$, at the pixel $(p_i,q_i)$. 

$$\mathbf{P}^\star = \lbrace (u_{1_i}^\star,G(p_i,q_i))^\mathrm{T} : u_{1_i}^\star \in u_1 \rbrace \in \mathbb{R}^{2\times n}$$


**Peak Path Finder Algorithm**

Upon deriving the set $\mathbf{P}^\star$, our next objective is to delineate a trajectory that traverses the 'peak' regions of this set. This trajectory is aimed at encapsulating the essential characteristics of each vector in $\mathbf{P}^\star$ while reducing the data complexity. 

To achieve this, we propose an algorithm that identifies critical points along the 'peak' trajectory. 

Initially, we establish a procedure to partition the curve into several segments. Consider the length of each segment to be $\Delta L_i$. The total number of segments, denoted as $n$, is determined by the condition that the sum of the lengths of all segments equals the arc length of the curve between two points $u_{1_1}$ and $u_{1_2}$. 

$$\sum_{i=0}^n \Delta L_i = \int_{u_{1_1}}^{u_{1_2}} \sqrt{1 + (\frac{d}{du_1}\theta^\mathrm{T}\mathbf{U})^2} du_1$$

Utilizing the determined number of segments $n$, we develop an algorithm designed to identify, within each segment $\Delta L_i$, a vector from the set $\mathbf{P}^\star$ that exhibits the maximum value of the function $G(p_i,q_i)$. 

The algorithm proceeds as follows:
        
> $f\to void$<br>
> for $i$ $\in$ $n$:<br>
> &nbsp;&nbsp;&nbsp;&nbsp;Define segment boundaries: $L_i$, $L_{i+1}$<br>
> &nbsp;&nbsp;&nbsp;&nbsp;Initialize: <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;maxValue 
> $\leftarrow -\infty$<br>
>  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;maxVector $\leftarrow \phi$ <br>
> &nbsp;&nbsp;&nbsp;&nbsp;for $\mathbf{v} \in \mathbf{P}^\star$ within $(L_i, L_{i+1})$:<br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if $G(p_i, q_i)$ of $\mathbf{v}$ > maxValue:<br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;maxValue $\leftarrow G(p_i, q_i)$ of $\mathbf{v}$<br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;maxVector $\leftarrow \mathbf{v}$<br>
> if maxVector $\neq \phi$ :<br>
> &nbsp;&nbsp;&nbsp;&nbsp;Add maxVector to the result set

### Result:
We applied the aforementioned algorithm for the cell shown in figure 7-2.

<div align="center">

![Start-up window](app/images_readme/curved_cell_18.png)  
</div>
<p align="center">
Fig.7-2: An image of a "curved" <i>E.coli</i> cell.
</p>

Figure 7-3 shows all the projected points on the center curve.

<div align="center">

![Start-up window](app/images_readme/projected_points.png)  
</div>
<p align="center">
Fig.7-3: All the points(red) projected onto the center curve(blue).
</p>


Figure 7-4 depicts the result of projection onto the curve.

<div align="center">

![Start-up window](app/images_readme/projected_points_18.png)  
</div>
<p align="center">
Fig.7-4: Projected points (red) onto the center curve.
</p>




Figure 7-3 describes the result of the peak-path finder algorithm.

<div align="center">

![Start-up window](app/images_readme/peak_path_18.png)  
</div>
<p align="center">
Fig.7-4: The estimated peak path by the algorithm.
</p>


# Data analysis

For data analysis, you are welcome to use your original scripts. (We provide how to connect to the database (i.e., *.db) in the different section.) 

However, you can also utilize the default scripts provided to review the cells you have labeled with the application.

The following *E.coli* cell in figure 8-1 is one of the output cells in the directory **"Cell/ph/"**.

<div align="center">

![Start-up window](app/images_readme/cell_results/ph.png)  
</div>

<p align="center">
Fig.8-1 A ph image of an <i>E.coli</i> cell.
</p>

## Output Folders/Files

### Cell/ph/

In this directory, the raw image(ph) of each cell with its contour(green) in the arbitrary set square is stored.  (e.g. fig.8-1)

The center of the cell is set to the center of the square.

### Cell/fluo1/
In this directory, the image(fluo1-channel) of each cell in the in the arbitrary set square is stored.

The center of the cell is set to the center of the square.

Figure 8-2 shows the fluo1 image of fig.8-1

<div align="center">

![Start-up window](app/images_readme/cell_results/fluo1.png)  
</div>

<p align="center">
Fig.8-2 A fluo1 image of the <i>E.coli</i> cell in fig.8-1.
</p>

### Cell/fluo1_incide_cell_only/

In this directory, only the cells(fluo1-channel) surrounded by the contour(red in fig.8-2) are stored.

The center of the cell is set to the center of the square.

Figure 8-3 shows areas inside the contour(red, fig.8-2).

<div align="center">

![Start-up window](app/images_readme/cell_results/incide_cell.png)  
</div>

<p align="center">
Fig.8-3 Areas surrounded by the contour(red, fig.8-2).
</p>


### Cell/gradient_magnitudes/

In this directory, each image has the gradient information of the fluorescence intensities inside the cell calculated by the Sobel operator, plotted on the coordinates of the original cell pixels.

The center of the cell is set to the center of the square.

Figure 8-4 shows the plotted gradient information.

<div align="center">

![Start-up window](app/images_readme/cell_results/gradient_magnitude0.png)  
</div>

<p align="center">
Fig.8-4 Calculated gradient at each pixel inside the cell.
</p>

### Cell/histo/
In this directory, histogram of the fluorescence intensities in 256 levels from 0 to 255 for each cell is stored. 

Figure 8-5 shows the histogram for the fluorescence information in fig.8-3.

<div align="center">

![Start-up window](app/images_readme/cell_results/histo.png)  
</div>

<p align="center">
Fig.8-5 Calculated gradient at each pixel inside the cell.
</p>


### Cell/histo_cumulative/
In this directory, a plotted cummulative freqency from Fluo. intensity 0  to 255 for each cell is stored.

Figure 8-6 shows the cumulative frequency plot for the cell.

<div align="center">

![Start-up window](app/images_readme/cell_results/histo_c.png)  
</div>

<p align="center">
Fig.8-6 Cumulative frequency plot for the cell.
</p>

### Cell/replot/

In this directory, the fluorescence information in each cell normalized between 0 and 1, is stored.
Note that the for the coordinates are in the new basis U.

Figure 8-7 shows the cumulative frequency plot for the cell.

<div align="center">

![Start-up window](app/images_readme/cell_results/replot.png)  
</div>

<p align="center">
Fig.8-7 Re-plotted fluo. information for the cell. 
</p>

The fluorescence information is normalized 0 to 1, then plotted with a colormap "inferno" as follows.

<div align="center">

![Start-up window](app/images_readme/colorbar.png)  
</div>

### Cell/unified/
In this directory, the combined images (ph and fluo1) are stored. This is for a demonstrating purpose.

Figure 8-8 shows the unified image of fig. 8-1 and fig.8-2.

<div align="center">

![Start-up window](app/images_readme/cell_results/unified.png)  
</div>

<p align="center">
Fig.8-8 The unified image of fig. 8-1 and fig.8-2. 
</p>


### Cell/projected_points/

This directory stores the result of the aforementioned projection algorithm for each cell.

Figure 8-9 shows the result of projection. 
<div align="center">

![Start-up window](app/images_readme/cell_results/projected_points.png)  
</div>

<p align="center">
Fig.8-9 The result of projection for the cell in fig.8-1. 
</p>

### Cell/peak_path/

In this directory, the estimated peak path for each cell using the aforementioned algoritm is stored.

Figure 8-10 depicts the estimated peak path for the cell in fig.8-1.

<div align="center">

![Start-up window](app/images_readme/cell_results/peak_path.png)  
</div>

<p align="center">
Fig.8-10 The result of the peak path finder algorithm for the cell in fig.8-1. 
</p>

### Cell/3dplot/
This directory contains the 3d plot for each cell. Z-axis represents the fluorescence intensity for each pixel incide the cell.

Figure 8-11 shows the 3d plot of the cell in fig. 8-2.

<div align="center">

![Start-up window](app/images_readme/cell_results/3dplot.png)  
</div>

<p align="center">
Fig.8-11 The 3d plot of the cell in fig. 8-2
</p>


# Quick Overview of the cells
After running the scripts in "data_analysis" mode, several .png files will be generated in the same directory as "cytomatic4.py".

Note that <u>"all" mode automatically includes this process.</u> so that you can directly check the results after running the processes.

In this section, we use "test_database.db" to demonstrate.


You can also run the scripts with the following commands to achieve the same results:

```bash
cd CytoMatic4
python3 demo.py
```
## PH
Displays all raw cells with their contours (in green) in a single image.

<div align="center">

![Start-up window](app/images_readme/test_database_ph.png)  
</div>


## Fluo
Shows all raw fluorescent cells with their contours (in red) in a single image.
<div align="center">

![Start-up window](app/images_readme/test_database_fluo1.png)  
</div>

## Replot 
Shows all the replot figures in a single image.
<div align="center">

![Start-up window](app/images_readme/test_database_replot.png)  

</div>

## Histo
Shows all the histogram figures in a single image.

<div align="center">

![Start-up window](app/images_readme/test_database_histo.png)  

</div>

## Histo Cumulative
Shows all the cumulative histogram figures in a single image.

<div align="center">

![Start-up window](app/images_readme/test_database_histo_cumulative.png)  

</div>

## Projected Points
Shows all the figures of the results for projection in a single image.

<div align="center">

![Start-up window](app/images_readme/test_database_projected_points.png)  

</div>


## Projected Points
Shows all the figures of the estimated peak paths in a single image.

<div align="center">

![Start-up window](app/images_readme/test_database_peak_paths.png)  

</div>


# Connecting to the database
You can connect to the database with the following scripts.


```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine = create_engine('sqlite:///database_name.db', echo=True)
Session = sessionmaker(bind=engine)
Base = declarative_base()
class Cell(Base):
    __tablename__ = 'cells'
    id = Column(Integer, primary_key=True)
    cell_id = Column(String)
    label_experiment = Column(String)
    manual_label  = Column(Integer)
    perimeter = Column(FLOAT)
    area = Column(FLOAT)
    img_ph = Column(BLOB) 
    img_fluo1 = Column(BLOB)
    img_fluo2 = Column(BLOB)
    contour = Column(BLOB)
    center_x = Column(FLOAT)
    center_y = Column(FLOAT)

with Session() as session:
    cells = session.query(Cell).all()
    for cell in cells:
        print(cell.cell_id, cell.manual_label)
        # Arbitrary image processing/analysis
```


# License
[OpenCV License](https://github.com/opencv/opencv/blob/master/LICENSE)

# Contributors
- Yunosuke Ikeda b182587@hiroshima-u.ac.jp (Lead Developer)
- Haruya Tsurusako e3131901@foreign.kitakyu-u.ac.jp (Debugging: Cell Rotation algorithms by  circumscribed quadrangle.(not used in this version.))
- Kyota Sanagi b200874@hiroshima-u.ac.jp(Debugging: Tkinter user interface)
- Syunsuke Komatsu m232298@hiroshima-u.ac.jp(Debugging: Tkinter user interface)
