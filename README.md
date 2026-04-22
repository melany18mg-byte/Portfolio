                             ATC4                           market_id     manufacturer  Product business_rule                 Pack    protection  ... log_price  market_size  log_market_size       HHI  lag_sales  is_brand  is_protected
0  A02B2 INIBITORI POMPA PROTONIC  esomeprazolo | ss sistemici solidi  aurora biofarma  abigerd         brand  compr gastr 20mg 14  mai protetto  ...  1.144223   8558869.96        15.962479  0.117538        NaN         1             1
1  A02B2 INIBITORI POMPA PROTONIC  esomeprazolo | ss sistemici solidi  aurora biofarma  abigerd         brand  compr gastr 20mg 28  mai protetto  ...  1.501853   8558869.96        15.962479  0.117538     869.78         1             1
2  A02B2 INIBITORI POMPA PROTONIC  esomeprazolo | ss sistemici solidi  aurora biofarma  abigerd         brand  compr gastr 40mg 14  mai protetto  ...  1.403643   8558869.96        15.962479  0.117538    7336.66         1             1
3  A02B2 INIBITORI POMPA PROTONIC  esomeprazolo | ss sistemici solidi  aurora biofarma  abigerd         brand  compr gastr 40mg 28  mai protetto  ...  1.761300   8558869.96        15.962479  0.117538     850.63         1             1
4  A02B2 INIBITORI POMPA PROTONIC  esomeprazolo | ss sistemici solidi  aurora biofarma  abigerd         brand  compr gastr 20mg 14  mai protetto  ...  1.144223   7672834.02        15.853197  0.111128    5884.02         1             1

[5 rows x 20 columns]
['ATC4', 'market_id', 'manufacturer', 'Product', 'business_rule', 'Pack', 'protection', 'month', 'product_share', 'eur', 'units', 'price', 'log_share', 'log_price', 'market_size', 'log_market_size', 'HHI', 'lag_sales', 'is_brand', 'is_protected']
                    units  price  market_size       HHI  is_brand  log_units  log_price  log_market_size  lag_units  log_lag_units
Product month                                                                                                                     
abigerd 2022-09-01   1634   4.49   8558869.96  0.117538         1   7.398786   1.501853        15.962479      277.0       5.624018
        2022-09-01    209   4.07   8558869.96  0.117538         1   5.342334   1.403643        15.962479     1634.0       7.398786
        2022-09-01   1011   5.82   8558869.96  0.117538         1   6.918695   1.761300        15.962479      209.0       5.342334
        2022-10-01    269   3.14   7672834.02  0.111128         1   5.594711   1.144223        15.853197     1011.0       6.918695
        2022-10-01   1314   4.49   7672834.02  0.111128         1   7.180831   1.501853        15.853197      269.0       5.594711
['Product', 'month']

================ FIXED EFFECTS - SOLO PRODOTTO ================

                          PanelOLS Estimation Summary                           
================================================================================
Dep. Variable:              log_units   R-squared:                        0.1091
Estimator:                   PanelOLS   R-squared (Between):             -1.7579
No. Observations:               49759   R-squared (Within):               0.1091
Date:                Wed, Apr 22 2026   R-squared (Overall):             -1.0652
Time:                        17:06:31   Log-likelihood                -8.693e+04
Cov. Estimator:             Clustered                                           
                                        F-statistic:                      1517.7
Entities:                         186   P-value                           0.0000
Avg Obs:                       267.52   Distribution:                 F(4,49569)
Min Obs:                       1.0000                                           
Max Obs:                       972.00   F-statistic (robust):             27.664
                                        P-value                           0.0000
Time periods:                     108   Distribution:                 F(4,49569)
Avg Obs:                       460.73                                           
Min Obs:                       161.00                                           
Max Obs:                       530.00                                           
                                                                                
                                Parameter Estimates                                
===================================================================================
                 Parameter  Std. Err.     T-stat    P-value    Lower CI    Upper CI
-----------------------------------------------------------------------------------
log_price           0.9964     0.1165     8.5562     0.0000      0.7681      1.2246
log_market_size     1.0972     0.1895     5.7901     0.0000      0.7258      1.4686
HHI                -1.7508     1.2900    -1.3573     0.1747     -4.2791      0.7775
log_lag_units       0.1604     0.0340     4.7187     0.0000      0.0938      0.2271
===================================================================================

F-test for Poolability: 141.88
P-value: 0.0000
Distribution: F(185,49569)

Included effects: Entity

================ FIXED EFFECTS - PRODOTTO + TEMPO ================

                          PanelOLS Estimation Summary                           
================================================================================
Dep. Variable:              log_units   R-squared:                        0.0684
Estimator:                   PanelOLS   R-squared (Between):              0.3874
No. Observations:               49759   R-squared (Within):               0.0863
Date:                Wed, Apr 22 2026   R-squared (Overall):              0.5941
Time:                        17:06:32   Log-likelihood                -8.427e+04
Cov. Estimator:             Clustered                                           
                                        F-statistic:                      908.30
Entities:                         186   P-value                           0.0000
Avg Obs:                       267.52   Distribution:                 F(4,49462)
Min Obs:                       1.0000                                           
Max Obs:                       972.00   F-statistic (robust):             176.61
                                        P-value                           0.0000
Time periods:                     108   Distribution:                 F(4,49462)
Avg Obs:                       460.73                                           
Min Obs:                       161.00                                           
Max Obs:                       530.00                                           
                                                                                
                                Parameter Estimates                                
===================================================================================
                 Parameter  Std. Err.     T-stat    P-value    Lower CI    Upper CI
-----------------------------------------------------------------------------------
log_price           1.0339     0.1118     9.2454     0.0000      0.8147      1.2531
log_market_size     0.7103     0.1698     4.1825     0.0000      0.3774      1.0432
HHI                -1.2896     1.3790    -0.9352     0.3497     -3.9924      1.4132
log_lag_units       0.0440     0.0333     1.3187     0.1873     -0.0214      0.1093
===================================================================================

F-test for Poolability: 118.99
P-value: 0.0000
Distribution: F(292,49462)

Included effects: Entity, Time

================ FIXED EFFECTS SEMPLICE ================

                          PanelOLS Estimation Summary                           
================================================================================
Dep. Variable:              log_units   R-squared:                        0.0665
Estimator:                   PanelOLS   R-squared (Between):              0.4747
No. Observations:               49759   R-squared (Within):               0.0735
Date:                Wed, Apr 22 2026   R-squared (Overall):              0.6608
Time:                        17:06:33   Log-likelihood                -8.432e+04
Cov. Estimator:             Clustered                                           
                                        F-statistic:                      1175.4
Entities:                         186   P-value                           0.0000
Avg Obs:                       267.52   Distribution:                 F(3,49463)
Min Obs:                       1.0000                                           
Max Obs:                       972.00   F-statistic (robust):             213.42
                                        P-value                           0.0000
Time periods:                     108   Distribution:                 F(3,49463)
Avg Obs:                       460.73                                           
Min Obs:                       161.00                                           
Max Obs:                       530.00                                           
                                                                                
                                Parameter Estimates                                
===================================================================================
                 Parameter  Std. Err.     T-stat    P-value    Lower CI    Upper CI
-----------------------------------------------------------------------------------
log_price           1.0029     0.1069     9.3821     0.0000      0.7933      1.2124
log_market_size     0.7005     0.1772     3.9529     0.0001      0.3532      1.0478
HHI                -1.2270     1.4398    -0.8522     0.3941     -4.0491      1.5951
===================================================================================

F-test for Poolability: 346.56
P-value: 0.0000
Distribution: F(292,49463)

Included effects: Entity, Time
