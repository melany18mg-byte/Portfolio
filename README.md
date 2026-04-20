
=== PRICE vs SHARE ===
                  price  product_share
price          1.000000       0.303909
product_share  0.303909       1.000000

=== BRAND vs INN ===
                 product_share     price
business_rule_x                         
Brand                 0.013578  4.247272
INN                   0.005931  3.568751

=== MODEL RESULTS PRAZOLI ===
C:\Users\GOMEZME1\OneDrive - Sandoz\Desktop\Portfolio project\.venv\Lib\site-packages\statsmodels\regression\linear_model.py:1966: RuntimeWarning: divide by zero encountered in scalar divide
  return np.sqrt(eigvals[0]/eigvals[-1])
                            OLS Regression Results                            
==============================================================================
Dep. Variable:              log_share   R-squared:                       0.082
Model:                            OLS   Adj. R-squared:                  0.082
Method:                 Least Squares   F-statistic:                     2222.
Date:                Mon, 20 Apr 2026   Prob (F-statistic):               0.00
Time:                        17:00:49   Log-Likelihood:            -1.1162e+05
No. Observations:               49945   AIC:                         2.232e+05
Df Residuals:                   49942   BIC:                         2.233e+05
Df Model:                           2                                         
Covariance Type:            nonrobust                                         
=======================================================================================
                          coef    std err          t      P>|t|      [0.025      0.975]
---------------------------------------------------------------------------------------
const                  -0.8559      0.213     -4.023      0.000      -1.273      -0.439
log_price               1.6240      0.027     59.284      0.000       1.570       1.678
is_brand                     0          0        nan        nan           0           0
months_since_launch    -0.0124      0.000    -36.541      0.000      -0.013      -0.012
==============================================================================
Omnibus:                     2410.489   Durbin-Watson:                   0.945
Prob(Omnibus):                  0.000   Jarque-Bera (JB):             2841.957
Skew:                          -0.535   Prob(JB):                         0.00
Kurtosis:                       3.468   Cond. No.                          inf
==============================================================================

Notes:
[1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
[2] The smallest eigenvalue is      0. This might indicate that there are
strong multicollinearity problems or that the design matrix is singular.
