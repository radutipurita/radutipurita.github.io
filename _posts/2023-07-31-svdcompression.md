---
title: SVD Image Compression
date: 2023-07-31 12:00:00 -500
categories: [mathematics,algebra,computerscience]
tage: [svd,image]
math: true
---

### Let's have fun with basic linear algebra

Today we are going to look into a simple application in image compression, one which uses the decomposition into singular values and the low rank approximation.

The singular value decomposition or simply the SVD is a very powerful tool of linear algebra which basically allows you to slice every matrix into three other matrices which have interesting properties. In other words, suppose we have a matrix $ A \in {R}^{m \times n }$, then we could represent it as the product of the following matrices:
$U \Sigma{V}^{T} $. $U \in {R}^{m\times m }$ and $V \in {R}^{n\times n}$ are both orthogonal matrices and $\Sigma \in {R}^{m \times n }$ is a diagonal matrix. $\Sigma$ has the form:

$$
 \begin{pmatrix}
  \sigma_1 & 0 & \cdots & 0 \\
  0 & \sigma_2 & \cdots & 0 \\
  \vdots  & \vdots  & \ddots & \vdots  \\
  0 & 0 & \cdots & \sigma_n 
 \end{pmatrix}$$
 
 where $\sigma_1,\sigma_2,... \sigma_{n} $ are called the singular values. Normally they are ordered such that $\sigma_1 \ge\sigma_2\ge... \ge\sigma_{n} $.

 There is a standardized method of obtaining the SVD which we are not going to go over as we are going to use the result provided by the Python library Numpy.

Sometimes when dealing with extremely big matrices we want to simply ignore the elements which do not contain a lot of information. This is where the low-rank approximation comes into good use. A rank $r$ approximation is one in which we keep only the first $r$ singular values and simply neglect the others. Calculating back the product of the three matrices we obtain an approximation $ {A}^{*} $ for the initial matrix $A$. For more insight one could look into the Eckart-Young theorem.

Now let's go into our example. We are going to try to compress an image by using the aforementioned method. The picture will be treated as a big matrix of pixels. The code written will use the python libraries OpenCV and Numpy.

```python
import cv2
import numpy as np
```

Now let's read the initial picture:

```python
image = cv2.imread("Carpathians.jpeg")
```


![Plot](/assets/img/sample/Carpathians.jpeg){: width="500" height="500" }

As we can observe this is a high resolution image. Let's try to compress it. Every pixel has three components:
 - The Red Channel
 - The Green Channel
 - The Blue Channel

 Thus, we need to split our image into three matrices each containing the values of its respective channel.
 ```python
 blue, green, red = cv2.split(image)
 ```

For the blue channel of the image let's calculate the corresponding three matrices: $U$, $\Sigma$ and $V$.

```python
blueU, blueS, blueV = np.linalg.svd(blue, full_matrices=False)
```
Here we pass the value False as we want to calculate the "Economy SVD" and make our program a little faster (as a side note it does not calculate the entire $U$ matrix).

After choosing the rank we want to approximate with, we simply use the following line of code for calculating the ${A}^*$ matrix while using the SVD's bold statement.
```python
blue_approx = blueU[:,:rank] @ blueS[:rank, :rank] @ blueV[:rank, :]
```
After doing the exact same thing for the other green and blue channels we only need to merge them.
```python
merged_approx = cv2.merge([blue_approx, green_approx, red_approx])
```
For different ranks we obtain images differently compressed:

![Plot](/assets/img/sample/compressed0.png){: width="500" height="500" }
![Plot](/assets/img/sample/compressed1.png){: width="500" height="500" }
![Plot](/assets/img/sample/compressed2.png){: width="500" height="500" }
![Plot](/assets/img/sample/compressed3.png){: width="500" height="500" }

If we were to plot the cumulative sum of the singular values we would see that in the beginning we gain a lot of information by just a slight increase in the value of the ranks. That being said though towards the end there is no significant gain in information.

$$\frac{\sum_{k = 1}^{r} \sigma_k}{\sum_{k=1}^{n}\sigma_k}$$

![Plot](/assets/img/sample/plot_sum.png){: width="500" height="500"}

Resources:
Brunton, S., & Kutz, J. (2022). Data-Driven Science and Engineering: Machine Learning, Dynamical Systems, and Control (2nd ed.). Cambridge: Cambridge University Press. doi:10.1017/9781009089517
