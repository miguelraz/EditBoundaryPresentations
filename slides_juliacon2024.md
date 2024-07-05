---
marp: false
title: editboundary_juliacon2024
paginate: true
theme: gaia
math: mathjax
backgroundColor: #fff

---

<style>
    section {
    font-size: 35px;
    line-height: 1.2;
    }
    section::after {
        content: attr(data-marpit-pagination) ' / '
        attr(data-marpit-pagination-total);
    }
</style>

<!-- _class: lead -->


### **EditBoundary.jl - A Quest to Revive Structured Quad Meshes**

by

##### <font color="blue">Miguel Raz Guzmán Macedo</font>

##### Pablo Barrera Sánchez

##### Iván Méndez Cruz

#### <font color="blue">UNAM</font>

## **JuliaCon 2024 Eindhoven, Netherlands** ![width:200px](./images/logo-SMCC-ENC.jpg)


---

### **Outline**

- **Introduction** 
- **Line Simplification**
- **Noise Detection and Reduction**
- **Method for Polygonal Approximation**
- **Conclusions and Future Work**

---
<!-- _class: lead -->

### **Introduction**
- Motivation
- Problem Statement
- The module <font color="blue">EditBoundary.jl</font>
- Contributions

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 30px;
        line-height: 1.2;
    }
</style>

### **Motivation:** Initial Step for Mesh Generation

<iframe
    width="1100"
    height="550"
    src="/home/mrg/oss/presentacion_enc2023_9sep/images/honda_lagoon.html"
></iframe>

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 30px;
        line-height: 1.2;
    }
</style>

### **Motivation:** Applications for Map Generalization

<iframe
    width="1000"
    height="500"
    src="http://gaia.inegi.org.mx"
></iframe>

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size:30px;
        line-height: 1.2;
    }    
    .columns {
        display: grid;
        grid-template-columns: repeat(2, minmax(0, 1fr));
        gap: 0.5rem;
        line-height: 1.2;                                                       
    }
</style>

### **Problem Statement**

<div class="columns">
<div>

Approximate a <font color="blue">polygonal contour</font> by a <font color="red">shape preserving polygon</font> with fewer points.

<font color="blue">The initial contour</font> may have high level of detail, noise or several holes.

<font color="red">The approximation</font>  preserves dominant points or encloses approximately the same area.

</div>
<div>

<iframe
    width="600"
    height="550"
    src="./images/amistad_simp_radius.html"
></iframe>

</div>
</div>

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 30px;
        line-height: 1.2;
    }
</style>

<!-- _class: lead -->

###  **Edit Boundary.jl**

- A module for polygonal approximation of 2D contours
- Coded in  <font color="purple">**Julia**</font> [[1],[3]](./slides_enc2023.html#23)
- Part of the mesher <font color="green">UNAMalla 6</font>
- <font color="red">See Demo in action!</font>
    
    Simplify the contour of <font color="blue">the water dam *La Amistad*</font> in the State of Coahuila.

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 26px;
        line-height: 1.2;
    }
    .columns {
        display: grid;
        grid-template-columns: repeat(2, minmax(0, 1fr));
        gap: 0.5rem;
        line-height: 1.2;
    } 
</style>

<!-- _class: lead -->

<div class="columns">
<div>

**Older versions of EditBoundary** [[2],[6],[8]](./slides_enc2023.html#23)

- Matlab (2012) <font color="green">
- UNAMalla</font> 5 (2014)


![width:500px](./images/amistad_collinear_test.svg)

<font color="red">**Issues in preserving the shape of contours with high level of detail**</font> such as the water dam *La Amistad*. 

</div>
<div>

**Before:**
- Simplification based on Pavlidi's collinearity test [[5]](./slides_enc2023.html#23) and Ray's dominant point detection [[7]](./slides_enc2023.html#23) 
- Smoothing by conic and tension splines

**After:**
- Simplification based on triangles measures
- Smoothing by constrained minimization of the perimeter

<font color="blue">**New Contributions:**</font>

- Radius Method for Simplification 
- Test for Noise Detection

</div>
</div>

---
<!-- _class: lead -->

### **Line Simplification**

- Simplification of polygons
- Area method
- Radius method

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size:24px;
        line-height: 1.2;
    }
    .columns {
        display: grid;
        grid-template-columns: repeat(2, minmax(0, 1fr));
        gap: 0.5rem;
        line-height: 1.2;                                                       
    }
</style>

### **Simplification of Polygons**

<div class="columns">
<div>

Given a polygon <font color="blue">$P$</font> and a tolerance, 
find the polygon <font color="red">$P'$</font> with the fewest vertices such that
- $\text{vertices}({\color{red}P'}) \subset \text{vertices}({\color{blue}P})$
- $\text{score}({\color{red}P'}) \leq  \text{tolerance}$
   
   E.g. $\quad\text{score}({\color{red}P'}) = 
    \frac{| \text{area}({\color{red}P'}) - 
    \text{area}({\color{blue}P})|}
    {\text{area}({\color{blue}P})}$

$\begin{array}{|r|r|}
 \hline 
 \textbf{Simplification} & \textbf{Score}\times 10^{3} \\
 \hline 
  19.5 \% & 2.37\times 10^{-3} \\
 \hline 
  46.8 \% & 2.08\times 10^{-2} \\
  \hline 
  89.7 \% & 4.35\times 10^{-1} \\
 \hline
\end{array}$

<b>Table:</b> Score for the simplification of the water dam *Vicente Guerrero*. The simplification level is the percentage of deleted points.

</div>
<div>

<iframe
    width="600"
    height="550"
    src="images/vicente_guerrero.html"
></iframe>

</div>
</div>

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 26px;
        line-height: 1.2;
    }
</style>

### **Simplification based on triangle measures**

Given a polygon $v_1,\dots,v_n$, form the triangles: 
$$T_i=v_{i-1}v_iv_{i+1}$$
and measure their areas 
$$a_i=\text{area}(T_i)$$

###  **Area method**

It is a sequential point elimination, also known as **Visvalingam-Whyatt algorithm** [[10]](./slides_enc2023.html#24) in Cartography.


**Idea:** At each step we find the triangle <font color="red">$T_j$</font> with the smallest area <font color="red">$a_j$</font> and delete the corresponding
vertex <font color="red">$v_j$</font>.

![bg right fit](images/metodo_areas.svg)

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 26px;
        line-height: 1.2;
    }
    .columns {
        display: grid;
        grid-template-columns: repeat(2, minmax(0, 1fr));
        gap: 0.5rem;
        line-height: 1.2;
    } 
</style>

###   **Area method**

<div class="columns">
<div>

**Implementation:** We use the average area $\overline{a}$ of the triangles so that the method be scale independent.


Given a threshold <font color="blue">$\epsilon$</font>,
find the triangle with the smallest area <font color="red">$a_j$</font>. If 
$${\color{red}a_j} < \overline{a}\cdot\color{blue}\epsilon,$$
then delete the vertex <font color="red">$v_j$</font>.

Repeat the above as long as the inequality holds.

Note that as the threshold increases, more vertices are removed.

</div>
<div>

 <iframe
    width="600"
    height="550"
    src="images/terminos_areas.html"
></iframe>
</div>
</div>

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 24px;
        line-height: 1.2;
    }

</style>

### **Radius Method** (new)

**Idea:**  Replace the areas of the triangles in the area method by the radii products:
$$\color{blue}2\cdot\text{circumradius}\times\text{inradius}$$

**Why?** By *Blundon's Inequality* [[11]](./slides_enc2023.html#23) every triangle with circumradius $R$, inradius $r$ and area $a$ satisfies:
$$a\leq 2Rr + 1.2r^2.$$
Non-obtuse triangles also satisfy
$$a\geq 2Rr + r^2$$
So if $r^2\approx 0$, then $\color{blue}a\approx 2Rr$.

Moreover, the above radii product can be computed using triangle sides
$\ell_1,\ell_2,\ell_3$:
$$\color{blue}2Rr = \ell_1\ell_2\ell_3/(\ell_1+\ell_2+\ell_3).$$

![bg right fit](images/radios.svg)

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 26px;
        line-height: 1.2;
    }
    .columns {
        display: grid;
        grid-template-columns: repeat(2, minmax(0, 1fr));
        gap: 0.5rem;
        line-height: 1.2;
    } 
</style>

###  **Radius Method**

<div class="columns">
<div>

**Implementation:** We use the average radii product $\overline{r}$ of the 
triangles so that the method be scale independent.

Given a threshold <font color="blue">$\epsilon$</font>, find the triangle with the smallest radii product  $\color{red}R_jr_j$. If 
$$ 2{\color{red}R_jr_j} < \overline{r}\cdot\color{blue}\epsilon,$$
then remove the vertex <font color="red">$v_j$</font>.

Repeat the above as long as the inequality holds.

</div>
<div>

 <iframe
    width="600"
    height="550"
    src="images/terminos_radios.html"
></iframe>
</div>
</div>

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 24px;
        line-height: 1.2;
    }
    .columns {
        display: grid;
        grid-template-columns: repeat(2, minmax(0, 1fr));
        gap: 0.5rem;
        line-height: 1.2;
    } 
</style>


<div class="columns">
<div>

### **Comparison and Experiments**

 - In both methods we can use the same range of values 
for the threshold $\color{green}\epsilon$ since 
$$\color{blue}\text{area}\approx 2\cdot\text{circumradius}\times\text{inradius}$$

- In the simplification of the Mexican water bodies from [AmeriGEOSS](https://data.amerigeoss.org/dataset/mexican-water-bodies) we have used 
    $$\color{green}0.1\leq\epsilon\leq 100$$


- In order to compare the polygonal approximations we use:
    $$\text{score}({\color{red}P'}) = 10^{3}\times
    \frac{| \text{area}({\color{red}P'}) - 
    \text{area}({\color{blue}P})|}
    {\text{area}({\color{blue}P})}$$
    
    The factor $10^{3}$ is introduced to get three significant digits

    We have generated shape preserving polygons with $\color{green}\text{score}\leq 1$ when $\color{green}\epsilon\leq 1$
</div>
<div>

$\begin{array}{|r|r|r|}
  \hline
  \text{threshold} & \text{simplification} & \text{score} \\\hline
  0.4 &  26.7 \% & 2.23\times 10^{-4} \\
  4   &  66.4 \% &  2.14\times 10^{-2} \\
  40  &  86.7 \% & 1.32\times 10^{-1} \\ \hline
\end{array}$

**Table 1:** Line simplification of the *Terminos Lagoon* using the radius method.


$\begin{array}{|r|r|r|}
  \hline
  \text{threshold} & \text{simplification} & \text{score} \\\hline
  0.4 &  48.3 \% &  4.55\times 10^{-3} \\
  4   &  72.3 \% &  1.09\times 10^{-2} \\
  40  &  87.2 \% &  8.67\times 10^{-2} \\ \hline
\end{array}$

**Table 2:** Line simplification of the *Terminos Lagoon* using the area method.
</div>
</div>

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 24px;
        line-height: 1.2;
    }
    .columns {
        display: grid;
        grid-template-columns: repeat(2, minmax(0, 1fr));
        gap: 0.5rem;
        line-height: 1.2;
    } 
</style>

### **Line simplification of the *Terminos Lagoon***

<div class="columns">
<div>
 <iframe
    width="500"
    height="500"
    src="images/terminos_area_score_radius.html"
></iframe>
<center>Area score for the radius method.</center>
</div>
<div>
 <iframe
    width="500"
    height="500"
    src="images/terminos_area_score_area.html"
></iframe>
<center>Area score for the area method.</center>
</div>
</div>
<br>

---
<!-- _class: lead -->

### **Approximation of Noisy Contours**

- Noise Detection
- Noise Reduction

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 26px;
        line-height: 1.2;
    }
    .columns {
        display: grid;
        grid-template-columns: repeat(2, minmax(0, 1fr));
        gap: 0.5rem;
        line-height: 1.2;
    } 
</style>

### **Noise Detection in a noisy contour** 
Sort the triangle areas in a log scale plot. **How do the areas decay?**

<iframe
    width="1100"
    height="450"
    src="images/mantarraya_areas.html"
></iframe>

<font color="blue">Note that the area plot of a noisy contour has a staircase shape with gaps.</font>

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 24px;
        line-height: 1.2;
    }
    .columns {
        display: grid;
        grid-template-columns: repeat(2, minmax(0, 1fr));
        gap: 0.5rem;
        line-height: 1.2;
    } 
</style>

### **Noise Reduction**

<div class="columns">
<div> 

We smooth the polygon by moving its vertices.

**How to move the vertices?** 

**Montanari's idea** [[3]](./slides_enc2023.html#21):
Given a polygon $\color{blue}v_1,\dots,v_n$ with perimeter $\rho$, and a threshold $\color{red}\delta$, find the minimum perimeter polygon $\color{green}u_1,\dots,u_n$ s.t.
$$\|{\color{green}u_i}-{\color{blue}v_i}\|_{\infty}
 \leq \rho\cdot{\color{red}\delta}, \quad i=1,\dots,n$$
- The smoothing is scale independent since the constraints depend on the perimeter $\rho$. 
- The threshold $\delta$ controls the smoothing. As it increases, the polygon becomes smoother.
- In the smoothing of digital contours from [MPEG7 dataset](http://congyang.de/downloads.html) we have used the following: 
    $$\color{red}10^{-5}\leq\delta\leq 10^{-3}$$

    to reduce noise and avoid over-smoothing

</div>
<div>
 <iframe
    width="600"
    height="550"
    src="images/mantarraya_smt.html"
></iframe>
</div>
</div>

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 26px;
        line-height: 1.2;
    }
    .columns {
        display: grid;
        grid-template-columns: repeat(2, minmax(0, 1fr));
        gap: 0.5rem;
        line-height: 1.2;
    } 
</style>

### **Method for Polygonal Approximation**

<div class="columns">
<div>

<ol>
 <li> Remove collinear points using the radius method.</li> 
 <li> Use the area plot for noise detection. </li>
 <li> Smooth noisy contours by minimizing the perimeter.</li>
 <li> Line simplification using the radius method.</li> 
</ol>

</div>
<div>
 <iframe
    width="600"
    height="550"
    src="images/flower_approx.html"
></iframe>
</div> 
</div>


---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 24px;
        line-height: 1.2;
    }
</style>

### **Conclusions**

Our method can get shape preserving polygons as we have tested by using *Editboundary.jl* and the inspection of the area score and area plots.


### **Future work** 
- Simplify contours of a region decomposition.
- Improvements for *Editboundary.jl*
    - performance
    - handle self-intersections
    - support for geospatial data format.

![bg right fit opacity:0.9](images/mapshaper_veracruz.png)

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 27px;
        line-height: 1.2;
    }
</style>

<br>
<br>
<br>
<br>

UNAMalla homepage: <font color="purple">tikhonov.fciencias.unam.mx</font>

Github: **EditBoundary.jl**

e-mail: <font color="blue">vanmc@ciencias.unam.mx</font>

![bg right fit opacity:0.8](images/golfo_barco.svg)

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 27px;
        line-height: 1.2;
    }
</style>

### **References**

1. [Bezanson, J et al.](./slides_enc2023.html#7) (2017) **Julia: a fresh approach to numerical computing.** *SIAM Review 59(1), 65-98*. <font color="purple">doi.org/10.1137/141000671</font>
2. [Carreón, C.](./slides_enc2023.html#8) (2008) **Un módulo para el tratamiento de contornos en el sistema UNAMALLA.** *Bachelor Thesis (spanish), UNAM, Mexico.* 
3. [Danisch & Krumbiegel](./slides_enc2023.html#7) (2021) **Makie.jl: Flexible high-performance data visualization for Julia.** *JOSS, 6(65), 3349.* <font color="purple">doi.org/10.21105/joss.03349</font>
4. [Montanari, U.](./slides_enc2023.html#18) (1979) **A note on minimal length polynomial approximation to a digitized contour.** *Commun. ACM 13(1), 41-47.*
5. [Pavlidis, T.](./slides_enc2023.html#8) (1982) **Algorithms for graphics and image processing.** Springer
6. [Ramírez, L. A.](./slides_enc2023.html#8) (2011) **Un módulo para el suavizamiento de contornos usando spline en tensión para el sistema Editboundary.** *Bachelor Thesis (spanish), UNAM, Mexico.*
7. [Ray & Ray](./slides_enc2023.html#8) (2013) **Polygonal approximation and scale-space analysis of closed digital curves.**
Apple Academic Press, Inc.
8. [Rivera, A.](./slides_enc2023.html#8) (1999) **Un punto de vista sobre las cónicas y su uso en el suavizamiento de polígonos.** *Bachelor Thesis (spanish), UNAM, Mexico.*

---

<!-- Scoped style -->
<style scoped>
    section {
        font-size: 27px;
        line-height: 1.2;
    }
</style>

### **References**

9. [UNAMALLA 4](./slides_enc2023.html#8) <font color="purple">lya.fciencias.unam.mx/unamalla/</font>
10. [Visvalingam, M., Whyatt, J. D.](./slides_enc2023.html#9) (1993) **Line generalisation by repeated elimination of points.** *Cartographic Journal, 30(1), 46-51.*
11. [Wu, S. H., Chu, Y. M.](./slides_enc2023.html#11) (2014) **Geometric interpretation of Blundon's inequality and Ciamberlini's inequality.** *J. Inequal. Appl. 381.*


