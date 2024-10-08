# polyfit
### Multivariate Polynomial Curve Fitting Sorcery for All Ages
#### Aka ( Polynomial Regression )

The goal of poplynomial regression is to find the best-fit curve to model the relationship between the independent variables (features) and a dependent variable (target). 

The example data included with this package  ( `./example_data.csv` ), will be used to explain 
- how to use this code to perform the afore mentioned sorcery
- what is happening ( to the best of my ability )

We will 
- create a new `FitMatrix` called `fm`
- load our example data into `fm.VectorSpace` 
    - `fm.LoadVectorSpaceFromCSV( path, nHeaders, nTargets )` 
- for each _Target_ ( $y_{n}$ column )
    - enter our `fm.FitOrders` 
    - and run the polynomial regression 
        - by calling `fm.CurveFit()` 
        - which wil in turn compute
            - `fm.ExponentMatrix` 
                - by calling `fm.ComputeExponentMatrix()` 
            - `fm.FeatureMatrix` 
                - by calling `fm.ComputeFeatureMatrix()`
            - `fm.InteractionSums` 
                - by calling `fm.ComputeInteractionSums()`
            - `fm.NormalMatrix` 
                - by calling `fm.ComputeNormalMatrix()`
            - `fm.Coefficients` 
                - by calling `fm.ComputeCoefficients()`
    - then we'll use the coefficients ( $\beta$ ) to 
        - predict the $\hat{y}$ values correponding to a given feature vector ( $i$ )
            - $\hat{y} =$ `fm.CalculateOutput( fm.VectorSpace[ i ].Feature )`
        - check our accuracey against the original corresponding _Target_ 
            - $y_{n} =$ `fm.VectorSpace[ i ].Target[ n ]`
            - $r = y_{n} - \hat{y} $

### example_data.csv
The example .csv file: `./example_data.csv` looks like this : 
```
y0,         y1,         y2,         x0,     x1,     x2
Degree_C,   kPa_Abs,    kPa_Diff,   T_ADC,  P1_ADC, P2_ADC
25.00000,   100.00000,  0.00000,    18112,  25519,  29222
25.00000,   100.00000,  50.00000,   18115,  25522,  58440
...
```
I have added an index column to the much prettier version below, so I can refer to individual rows in this documentation.

---
For my fellow **_applied folk_** 

This example data is the imaginary calibration input data for an imaginary digital flow sensor which uses Analog to Digital Converters ( ADCs ) to measure and spit out the values in the $x$ columns:
- $x_{0}$ is the ADC output from a temperature sensor 
    - outputs an integer value corresponding to the temperature 
    - output at a given temperature is unlikely to be effected by changes in either of the pressure conditions

- $x_{1}$ is the ADC output from an absolute pressure sensor 
    - outputs an integer value corresponding to the absolute pressure
    - output at a given absolute pressure will vary as temerature changes

- $x_{2}$ is the ADC output from a differential pressure sensor
    - outputs an integer value corresponding to the difference between pressures measured at two locations
        - location one is the same as the $x_{1}$ sensor
        - location two is behind some restriction where the pressure will always be equal to or lower than $x_{1}$, depending on the flow rate of a fluid passing through the device  
    - output at a given differential pressure will vary as absolute pressure changes
    - output at a given differential pressure will vary as temerature changes


These ADC outputs are useless until we calibrate the device, and to do that, we
- Subjected the devicec to a range of expected conditions

- Recorded the ADC outputs in the $x$ columns ( _Features_ ) 

- Recorded the actual applied conditions in the $y$ columns ( _Targets_ )
    - $y_{0}$ :  We applied $3$ temperatures 
        - 25, 50, and 75 °C
    - $y_{1}$ : At each temperature, we appliead $3$ absolute pressures
        - 100, 1000, and 2000 kPa
    - $y_{2}$ : At each absolute pressure, we appliead $3$ differential pressures 
        - 0, 50, and 100 kPa


Luckily our device is imaginary so I didn't have to do all of that data colletion and instead, just made up some data points I think might work.  ( If they don't work I'll just make up different data until it does work and leave you believing I nailed first try )

And BAM! Now we can use polynomial regression to create a set of coefficients for each of the $y$ values ( _Targets_ ). Each set of coefficients will allow us to use our ADC outputs, to calculate ( _predict_ ) the corresponding $y$ value at **any**
- Temperature between 25 and 75 °C
- Absolute pressure between 100 and 2000 kPa
- Differential pressure between 0 and 100 kPa
- Yes, even the many combinations of temperatures, pressures and differentials not present in our original input data.

---

$$ \begin{matrix} 
 & y_{0} & y_{1} & y_{2} & | & x_{0} &x_{1} & x_{2} \\\
 &  &  &  &  &  &  &  \\\
 & °C & kPa_{Abs} & kPa_{Diff} & | & T_{ADC} & P1_{ADC} & P2_{ADC} \\\
 &  &  &  &  &  &  &  \\\
0: & 25 & 100 & 0 & | &18112 &	25519 &	29222 \\\
1: & "  & "  & 50 & | &18115 &	25522 &	58440 \\\
2: & "  & "  & 100 & | &18118 &	25525 &	87658 \\\
3: & "  & 1000 & 0 & | &18115 &	49690 &	29225 \\\
4: & "  & "  & 50 & | &18118 &	49693 &	58443 \\\
5: & "  & "  & 100 & | &18121 &	49696 &	87661 \\\
6: & "  & 2000 & 0 & | &18118 &	76547 &	29227 \\\
7: & "  & "  & 50 & | &18120 &	76550 &	58445 \\\
8: &  " & "  & 100 & | &18123 &	76553 &	87663 \\\
 &  &  &  &  &  &  &  \\\
9: & 50 & 100 & 0 & | &36219 &	25521 &	29224 \\\
10: & "  & "  & 50 & | &36222 &	25523 &	58442 \\\
11: & "  & "  & 100 & | &36225 &	25526 &	87660 \\\
12: & "  & 1000 & 0 & | &36222 &	49692 &	29227 \\\
13: & "  & "  & 50 & | &36225 &	49695 &	58445 \\\
14: & "  & "  & 100 & | &36228 &	49698 &	87663 \\\
15: & "  & 2000 & 0 & | &36225 &	76549 &	29229 \\\
16: & "  & "  & 50 & | &36227 &	76551 &	58447 \\\
17: &  " & "  & 100 & | &36230 &	76554 &	87665 \\\
 &  &  &  &  &  &  &  \\\
18: & 75 & 100 & 0 & | &54326 &	25522 &	29226 \\\
19: & "  & "  & 50 & | &54329 &	25525 &	58444 \\\
20: & "  & "  & 100 & | &54332 &	25528 &	87662 \\\
21: & "  & 1000 & 0 & | &54329 &	49694 &	29228 \\\
22: & "  & "  & 50 & | &54332 &	49696 &	58446 \\\
23: & "  & "  & 100 & | &54335 &	49699 &	87664 \\\
24: & "  & 2000 & 0 & | &54332 &	76550 &	29231 \\\
25: & "  & "  & 50 & | &54334 &	76553 &	58449 \\\
26: &  " & "  & 100 & | &54337 &	76556 &	87667 \\\
\end{matrix} $$

---
### Loading Input Data
The example data has  
- $6$ columns
    - The first $3$ are $y$ columns ( Targt Values ) `N_TARGETS = 3`
    - The remaining $3$ are $x$ columns ( Feature Values )

- $29$ rows
    - $2$ are header rows `N_HEADERS = 2`
    - $27$ are data rows, and will be loaded into our `FitMatrix`, ( $0$ through $26$ above ). 
    Each data row will be used to create a pair of vectors.
    Row $0$ will be loaded as follows:

`fm.VectorSpace[0].TargetVector`

$$\begin{matrix} y_{0}: & 25 \\\ y_{1}:  & 100 \\\ y_{2}: & 0 \\\ \end{matrix}$$ 

`fm.VectorSpace[0].FeatureVector`

$$\begin{matrix} x_{0}: & 18112 \\\  x_{1}: & 25519 \\\ x{2}: & 29222 \\\ \end{matrix}$$

---
### Sum of Squared Errors (SSE) $\quad S(\beta) = \sum_{i=1}^{m} ( y_i  -  \hat{y_i} )^2$
The least squares method minimizes the sum of the squared differences (errors) 
between the observed values and the values predicted by the model. 

$m: \quad$ is the number of data points ( _Samples_  )

$y: \quad$ is the vector of observed values ( _Targets_  )

$y_i: \quad$ is  a specific _Target_ in the vector $y$

$\hat{y}: \quad$ is the vector of _predicted values_ ( $\hat{y} = X\beta$ ) 

$\hat{y_i}: \quad$ is a specific _predicted value_ corresponding to a given $y_i$

$X: \quad$ is the `FeatureMatrix`, where each row is an observation and each column is a feature.

$\beta: \quad$ is the vector of _Coefficients_ we will use to make predictions.

---
`FitMatrix.ComputeNormalMatrix()`
### The Normal Equation 
### $$\quad X^T X\beta = X^T y$$

$X^T\quad$ the **_Transpose_** of the matrix $X$.

If matrix $X$ is 

$$\begin{bmatrix} a & b & c \\\ d & e & f \\\ g & h & i \end{bmatrix} $$

Matrix $X^{T}$ is 

$$\begin{bmatrix} a & d & g \\\ b & e & h \\\ c & f & i \end{bmatrix} $$

$X^TX\quad$ the `NormalMatrix`, often called the _Gram Matrix (denoted $G$)_, a positive semi-definite, symmetric matrix of inner products of the columns of 𝑋:

- Positive Semi-Definite means all eigenvalues of $X^T X$ are positive

- Symmetric means $(X^TX)^T = 𝑋^𝑇 𝑋$ 

The *Normal Equation* is derived from the *Sum of Squared Errors* equation, written in: 
#### SSE Quadratic Form 
### $$S(\beta)  =  ( y - X\beta )^{T}  \quad  ( y - X\beta )$$

---
### 1. First we expand the quadratic form 

$S(\beta)= \quad y^{T}y\quad -\quad  y^{T}X\beta\quad -\quad X^{T}\beta^{T}y\quad+\quad X^{T}\beta^{T}X\beta$

- (2nd term) multiplying the **Transpose** of **$y$** with **$X$** and **$\beta$** in their original form is the same as 

- (3rd trem) multiplying **Transposed** **$X$** and **$\beta$** with **$y$** in its original form so...

#### SSE Expanded Quadratic Form 
### $$S(\beta) =  y^{T}y -2y^{T}X\beta + X^{T}\beta^{T}X\beta$$

- $y^{T} y\quad$ is the dot product of $𝑦$ with itself, resulting in a scalar representing the sum of the squared observed (**Target**) values.

- $−2𝑦^{T}𝑋\beta\quad$ is a linear term in $\beta$ representing the interaction between the observed  (**Target**) values and the predictions.

- $\beta^{𝑇} 𝑋^{T} 𝑋\beta\quad$ is a quadratic term in $\beta$, capturing the relationships between the features. It represents the model's predictions, squared and summed over all data points.

In this form the terms involving **$\beta$** are separated, allowing us to take the derivative and minimize the function.

---
### 2. We Take the Derivative with Respect to $\beta$

We differentiate each of the three terms in $𝑆 (\beta)$
 
$y$ does not depend on $\beta$

$$\frac{d}{d\beta}y^{T}y = 0$$

we’ve transposed $X$ because of the **matrix calculus rule** (we’re differentiating a dot product).

$$\frac{d}{d\beta}(-2y^{T}X\beta) = -2X^{T}y$$

quadratic term rule ...

$$\frac{d}{d\beta}(\beta^{T} X^{T} X\beta) = 2X^{T} X\beta$$

#### Derivative with Respect to $\beta$ 
### $$\frac{d}{d\beta}S(\beta) = -2X^{T}y + 2X^{T} X\beta$$

---
### 3. Set the Derivative Equal to Zero

We set the derivative equal to zero, to minimize $S(\beta)$

$$-2X^{T}y + 2X^{T} X\beta = 0$$

We simplify by 
- adding $-2X^{T}y$ to both sides 
- deviding the whole mess by $2$

... and we arrive at **The Normal Equation**

### $$\quad X^T X\beta = X^T y$$

---
`FitMatrix.ComputeCoefficients()`
## Solve The Normal Equation for $\beta$ ( **Coefficients** )
We devide both sides by $X^{T}X$ to isolate $\beta$

### $$\beta = (X^{T}X)^{-1} X^{T}y$$

So rearanged, the Normal Equation gives us the vector of `Coefficients` $\beta$ 
- which minimizes the sum of squared errors.

---
