# Elaboración de un modelo de pronóstico de ventas de productos de retail – Teamcore

Luciano Luca Fabbri Soto-Aguilar
<br/><br/>
*Concepción, Chile*
*2020*



##### En el presente trabajo se elabora un modelo de pronóstico de ventas de 413 productos en 7 cadenas comerciales basado en un modelo de boosting de gradiente de efectos mixtos auto-rezagado hasta siete días que considera el efecto idiosincrático de cada producto como efecto aleatorio y los efectos estacionales y promocionales como efectos fijos no lineales. El modelo propuesto fue parametrizado mediante una búsqueda manual de hyperparámetros a través de una validación cruzada del entrenamiento del modelo con datos de abril hasta octubre de 2019 con el pronóstico de las observaciones de noviembre de 2019 para finalmente ser evaluado con las observaciones de diciembre de 2019. El modelo obtuvo un R^2 y un MAPE fuera de muestra ex-post de 0.9 y 60% respectivamente, valores de  R^2 fuera de muestra superiores al 0.8 para todos los horizontes de pronóstico menores a 30 días y valores MAPE en torno al 20% para horizontes de pronóstico menores a 18 días.


###### © 2020 Luciano Luca Fabbri Soto-Aguilar

#### [Explore the code>>](https://github.com/lutifabbri/Capital-Structure-Determinants/blob/main/Capital%20Structure%20Determinants.ipynb)

<br/><br/>

### **1.   Metodología**

Para la elaboración de este proyecto se trabajó sobre datos correspondientes a mediciones temporales de ventas de 413 productos en 7 cadenas comerciales de abril 2019 a diciembre 2019. La información contenida representaba valores de ventas en términos unitarios y de ingresos junto con información promocional general. De esta forma la base de datos correspondía a un total de 1000 series temporales multivariantes en la que cada observación correspondía a una medición temporal de lo descrito anteriormente para un producto en una cadena comercial, lo que representó un total de 251,358 observaciones.
En primera instancia se realizó un análisis exploratorio de los datos para poder determinar y comprender de manera más profunda la naturaleza de las ventas que se modelaron mediante el modelo propuesto. Esto incluyó la identificación de características esenciales de las series de tiempo de los diferentes productos en diferentes cadenas comerciales, la relación cruzada entre el comportamiento de las ventas de un mismo producto en diferentes cadenas y otros aspectos que se consideraron relevantes de acuerdo con los datos disponibles. Una vez conocida la naturaleza de los datos y en función de las limitaciones de diferentes modelos se formuló la propuesta para el modelo de pronóstico de ventas.
Para evaluar el desempeño del modelo, éste fue parametrizado mediante una búsqueda manual de hyperparámetros a través de una validación cruzada del entrenamiento del modelo con datos de abril hasta octubre de 2019 con el pronóstico de las observaciones de noviembre de 2019 para finalmente ser evaluado con las observaciones de diciembre de 2019. Luego se evaluó el desempeño del modelo en datos de entrenamiento y de testeo mediante la utilización de las métricas R^2 y MAPE. Finalmente se hace un análisis exploratorio para determinar de forma más integral el desempeño del modelo y posibles alternativas para mejorar su rendimiento.


## **2.   Resultados**

*2.1   Análisis Exploratorio*

Se estimó que el pronóstico de ventas puede ser analizado desde dos perspectivas en términos generales: desde la perspectiva de gestión logística o cadena de suministro, o desde el punto de vista de la estrategia o inteligencia de negocios. Desde la primera perspectiva mencionada se considera que hacer el pronóstico de las unidades a vender es más adecuado, mientras que para la segunda se considera más razonable hacer el pronóstico de las ventas [CLP], esto sobre la suposición de que los valores ‘ventas_clp_dia’ correspondían a la utilidad (dado que la definición de este dato es ambigua). Dado que el efecto promocional puede incrementar de forma drástica las unidades vendidas sin necesariamente incrementar las utilidades (dado a que las promociones afectan, generalmente, la utilidad unitaria de los productos) se considera que el segundo enfoque es más útil para los objetivos de la empresa y por lo tanto se trabaja sobre éste.
La proporción de observaciones en períodos normales y de promoción es bastante balanceada, correspondiendo a un 54% y 46% de las observaciones respectivamente. Para evaluar la diferencia de medias de venta se aplicó la prueba no paramétrica U de Mann-Whitney la que indicó una diferencia estadística significativa. Sin embargo, esto no necesariamente indica que este aumento se debe particularmente al efecto promocional. Este efecto puede ser causado por la coincidencia temporal de las promociones con segmentos estacionales de alta demanda o debido a que promociones recientes estén afectando a productos con efectos significativos de tendencia, por lo que, en estos casos, el aumento de las ventas estaría más asociado a efectos temporales de las series.
En referencia a las cadenas comerciales, las cadenas 1,2 y 3 son aquellas que poseen la mayor cantidad de productos únicos. Por otra parte, desde abril a diciembre de 2019, la mayoría de los productos en las diferentes cadenas estuvieron sujetos a entre 3 y 5 promociones diferentes. De forma similar, cada promoción afectó en promedio a 73 productos distintos, lo que indica que las promociones debiesen ser bastante generales y no diseñadas en función de productos particulares. Es importante recalcar que, dado que no se tenía el detalle de las promociones, no fue posible analizar la elasticidad de la demanda de los productos, por lo que se estuvo restringido a un enfoque más generalizado respecto al impacto de las promociones en las ventas.
Mediante el análisis individual de cada serie, se observa que éstas están altamente correlacionadas en términos cruzados con sus pares de diferentes cadenas comerciales. Junto con esto y a grandes rasgos se observa que la gran mayoría de las series tienden a ser estacionales con una periodicidad mensual. Por otra parte, se observa que el comportamiento entre productos es bastante heterogéneo. En términos del número de observaciones, la gran mayoría de las series se compone de aproximadamente 250 observaciones, mientras la mayoría de los productos posee entre 250 y 750 observaciones. Dado que el número de observaciones por serie y por producto es reducido, la utilización de modelos individuales (por serie/producto) no es recomendada puesto que éstos debiesen tender a sobreajustarse a menos que sean parametrizados de tal manera de configurar modelos de muy baja varianza y alto sesgo.

*2.2   Formulación del Modelo*

En primera instancia se pensó en utilizar modelos estadísticos tradicionales como modelos ARMA, ARIMA, SARIMA, etc. para tener un benchmark de desempeño, sin embargo dada la configuración del problema, se considera que la implementación de éstos es infactible; para una aplicación sistematizada de estos modelos (uno por cada serie) sería necesaria la incorporación de un proceso manual para identificar los modelos apropiados para cada serie además de un proceso iterativo de transformaciones correspondientes de acuerdo al modelo seleccionado y la serie en cuestión, por lo que este enfoque fue descartado.
Considerando lo anterior se propone utilizar un
[modelo de efectos mixtos basado en boosting de gradiente](https://github.com/manifoldai/merf) auto-rezagado hasta siete días que considera el efecto idiosincrático de cada producto como efecto aleatorio y los efectos estacionales y promocionales como efectos fijos no lineales. Este modelo permite el ajuste de un modelo global, sin necesidad de clusterizar o separar las series según su comportamiento o según el producto. Las variables utilizadas son la siguientes:



<br/><br/>
##### *Tabla 1. Definición de Variables*
|Variable|Abreviación|Descripción|
|:-------|:-----------|:----------|
|Ventas<sub>ijt</sub>|V<sub>ijt</sub>|	Utilidades por venta de productos i en la cadena j en el período t|
|Es_promo<sub>ijt</sub>|P<sub>ijt</sub>|1, si el producto i en la cadena j está en promoción, 0 en otro caso|
|Día_semana_1<sub>t</sub>|DS_1<sub>t</sub>|1, si el período t corresponde al primer día de la semana, 0 en otro caso|
|...|	...|	...|
|Día_semana7<sub>t</sub>|DS_7<sub>t</sub>|1, si el período t corresponde al séptimo día de la semana, 0 en otro caso|
|Día_mes_1<sub>t</sub>|DM_1<sub>t</sub>|1, si el período t corresponde al primer día del mes, 0 en otro caso|
|...|...|...|
|Día_mes_1<sub>t</sub>|DM_1<sub>t</sub>|1, si el período t corresponde al primer día del mes, 0 en otro caso|

<br/><br/>

El modelo propuesto es el siguiente:

![model formulation](https://github.com/lutifabbri/Teamcore-Challenge/blob/main/Formulaci%C3%B3n_modelo.PNG)

Donde f(X) es la función desconocida estimada utilizando XGBoost, θ es el vector de hiperparámetros correspondiente que parametriza al modelo XGBoost, Z<sub>i</sub> es la matriz de covariables de efectos aleatorios b<sub>i</sub> es el vector de efectos aleatorios para el producto i y ϵ<sub>ijt</sub> el error aleatorio.



Sequential approach:
<br/><br/>
![Formulation1](https://github.com/lutifabbri/Capital-Structure-Determinants/blob/main/Dataset%20Images/Formulation1%20formula.png)

Static approach:
<br/><br/>
![Formulation2](https://github.com/lutifabbri/Capital-Structure-Determinants/blob/main/Dataset%20Images/Formulation2%20formula.png)


Where f(X) is the unknown function estimated using an XGBoost, Br<sub>it</sub>, Ch<sub>it</sub>, Mx<sub>it</sub>, Pe<sub>it</sub> are dichotomic variables that indicate the country of headquarters of each firm, θ is the corresponding hyperparameter vector that parameterizes XGBoost models, Z<sub>i</sub> is the covariate matrix of random effects, b<sub>i</sub> is the vector of random effects for company i ,  and ϵ<sub>it</sub> is the random error term. [SHAP Values](https://github.com/slundberg/shap#citations) were used to interpret the results of each model. The computing of this values does not consider firm-specific effects on leverage, leaving only the fixed effects of the corresponding explanatory variables which facilitates the exploration of  ‘pure’ effects of capital structure determinants within the framework of the objectives of this work.

The sequential formulation enabled to examine a more continuous variation of the effects of the different variables on leverage, however, a drawback in terms of interpretability is introduced because of the repetition of each variable caused by the lagged effect. The effects on firm leverage of each financial explanatory variable were plotted in a 3d graph with dimensions of variable value, year of the corresponding model and impact on firm leverage. On the other hand, formulation two only enables to plot 2d plots for each macro time period. Lastly the effects of each variable and formulation were compared according to the estimated effect on leverage according to the different capital structure theories and between the two formulations.


## **3.   Results**

An increase in model performance is achieved by including random firm effects which indicates the existence of non-negligible random effects. In other words, individual/internal firm factors have strong effects on leverage, which could be due to internal company policies or stakeholder management preferences. In terms of training and testing accuracy the sequential formulation shows satisfactory performance (see Figure 1), achieving an R2 testing score over 0.77 for 75% of the models and a minimum R2 training score of 0.96. Similarly, the static formulation also achieved satisfactory performance, with an average training and testing R2 score of 0.94 and 0.77, respectively. Respect to the performance of the models, it is possible to observe a positive trend between number of training data on the performance of the model in unobserved data (see Figure 1). It is possible to assume that obtaining new or more observations could result in an increase in performance. However, considering that the only way that the author of this project has to increase the volume of data is by imputing missing data under some criteria, and given the complex nature of the data used (panel data - time series for each variable per company) it is estimated that the use of any imputation technique could result in the addition of unnecessary noise instead of rich and meaningful information to explain firm leverage.

![Combined figure](https://github.com/lutifabbri/Capital-Structure-Determinants/blob/main/Dataset%20Images/Combined.png)

##### *Figure 1. Training samples and test score & R2 train/test KDE Joint distribution – Sequential Formulation.*
<br/><br/>



In aggregate terms the static and sequential formulations achieved practically equal performance. However, because changes in the effects of different variables on total firm leverage might occur in short time spans (few samples with altered behavior), this could possibly have non-significant influence in the performance metric (R2 score). This phenomenon of short time span changes in effect of variables could not possibly be detected by the static model formulation and would not be reflected in the aggregate performance of the model. Because of this, is encouraged to use both model formulations with these considerations in mind to get accurate inferences about the impact and change of different firm characteristics on total leverage. 

In terms of the effects of the different variables on firm leverage: for the sequential formulation, independent of the lag of the variable, the effect on leverage is the same, and both formulations show the same trend effects of each variable on firm leverage. It is encouraged to the reader to check the SHAP plots in this repository and to use the plotting tool in the notebook to explore interactively the effects of the different variables and how they vary across time (see gif 1).

![gif](https://github.com/lutifabbri/Capital-Structure-Determinants/blob/main/Variable%20effects%20-%20gifs/Formulation4%20-%20Total%20leverage%20-%20Liquidity%20-%20lag1.gif)
##### *Gif 1. Surface fitting to SHAP Values and crisis period display - Liquidity - lag1.*
<br/><br/>

From both model formulations, the following results are obtained; liquidity and firm size indicators are those that explain, on average, the highest proportion of firm leverage.  On the other hand, all variables presented both positive and negative effects on leverage depending on the value of the indicator and none presented an effect that could have been considered linear. The trends presented are sufficiently similar between periods to consider that the global financial crisis did not have a significant effect on the effect of the different financial indicators on the leverage of the companies.

Considering the expected effects of each variable according to Zeitun et. al. (2017), the results suggest that one theory is not predominant over another because the effect of the different determinants of the capital structure can be positive or negative depending on the values they take. However, it is possible to suggest that one theory better explains the capital structure for certain types of firms better than the other. In this way,  and in general terms, for larger and unprofitable companies, the Trade-off theory explains in a better way the debt structure of firms and the Pecking Order theory better explains the capital structure of smaller, more profitable, less tangible and more liquid firms.

## **4.   References**

[1] 	S. Pepur, M. Ćurak y K. Poposki, «Corporate capital structure: the case of large Croatian companies,» Economic research-Ekonomska istraživanja, pp. 498-514, 2016.

[2] 	S. Valcacer, H. J. de Moura, D. F. Lopes y V. A. Sobreiro, «Capital structure management differences in Latin American and US firms after 2008 crisis,» Journal of Economics, 2017.

[3] 	B. Harrison y T. W. Widjaja, «The determinants of capital structure: Comparison between before and after financial crisis,» Economic Issues, pp. 55-80, 2014.

[4] 	S. Myers, «Capital Structure Puzzle,» National Bureau of Economic Research, 1984.

[5] 	S. C. Myers y N. S. Majluf, «Corporate Financing and Investment Decisions When Firms Have InformationThat Investors Do Not Have,» National Bureau of Economic Research, 1984.

[6] 	P. Proença, R. M. S. Laureano y L. M. S. Laureano, «Determinants of capital structure and the 2008 financial crisis: evidence from Portuguese SMEs.,» Procedia-Social and Behavioral Sciences, pp. 182-191, 2014.

[7] 	R. Zeitun, A. Temimi y K. Mimounu, «Do financial crises alter the dynamics of corporate capital structure? Evidence from GCC countries.,» The Quarterly Review of Economics and Finance, pp. 21-33, 2017.

[8] 	S. Vodwal, V. Bansal y P. Sinha, «Impact of Financial Crisis on Determinants of Capital Structure of Indian Non-financial Firms: Estimating Dynamic Panel Data Model using Two-Step System GMM.,» Munich Personal RePEc Archive, 2019.

[9] 	S. Beyer, «Capital Structure of German publicly listed Firms: Evidence from the Financial Crisis. (Tesis de Licenciatura, University of Twente),» 2018.

[10] 	A. Iqbal y O. Kume, «Impact of financial crisis on firms’ capital structure in UK, France, and Germany,» Multinational Finance Journal, pp. 249-280, 2014.

[11] 	T. H. Trinh y N. T. Phuong, «Effects of financial crisis on capital structure of listed firms in Vietnam,» International Journal of Financial Research, pp. 66-74, 2016.

[12] 	J. Jermias y F. Yigit, «Factors affecting leverage during a financial crisis: Evidence from Turkey,» Borsa Istanbul Review, pp. 171-185, 2018. 

## **5.   Apendix**
**Appendix 1**: Impact of the different capital structure determinants on firm leverage according to static and sequential formulations. Pre-crisis, crisis, post-crisis estimated effects for the static formulation are first presented followed by a fixed view of the effects of the corresponding variable for the sequential formulation.

![Appendix1](https://github.com/lutifabbri/Capital-Structure-Determinants/blob/main/Dataset%20Images/Appendix1.png)

**Appendix 2**: Fixed views of the capital structure determinants are displayed to analyze the variation of their effect on firm leverage in the 2008 global financial crisis which is marked by two red planes.

![Appendix2](https://github.com/lutifabbri/Capital-Structure-Determinants/blob/main/Dataset%20Images/Appendix2.png)








