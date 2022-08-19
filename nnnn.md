Premissas
1. Manter o Dataframe somento com 'id' únicos. Classificação da tabela por 'date' decrescente e eliminação dos 'id' repetido.
2. As colunas 'sqft_living15' e 'sqft_lot15' não serão analisadas portado vamos excluir-las do dataframe.
3. Elaborar mais 02 colunas ('price/sqft_livig', 'price/sqft_lot'). O que vai dar um parametro mais proximo do real do valor.
4. Elaboração de tabela de Estatística Descritiva com as seguintes colunas ('price', 'bedrooms', 'bathrooms', 'sqft_living', 'sqft_lot', 'floors', 'sqft_above', 'sqft_basement', 'price/sqft_livig', 'price/sqft_lot') para analise, de possiveis outlier. Os outliers serão eliminados conforme analise de Q1 e Q3 sendo considerado um fator de 3 FIQ.
5. Elaboração de diagrama de caixa para observar os outliers das colunas ('price', 'bedrooms', 'bathrooms', 'sqft_living', 'sqft_lot', 'floors', 'sqft_above', 'sqft_basement','price/sqft_livig', 'price/sqft_lot').

6. 





1.A Haviam 21613 registros e foram removidos 177 (registros duplicados) passando a ter 21436.
2.Remoção do número de quartos 33 as outras caracteristicas não coencidem com esta quantidade de quartos. 21435
3.Remoção de terrenos onde a área do lote é menor que a área do living foram mantidados igual o superior. 20646

4.Filtro por zipcode estabeleceu as medianas por zipcode e foi fita uma recomendação foi filtrado todos os dados com recomendação 'yes'. 3873



```python
import pandas as pd
import seaborn as sns
import numpy as np
import matplotlib.pyplot as plt
from matplotlib import gridspec
```


```python
def get_data (path):
    data = pd.read_csv(path)
    
    return data
```


```python
def statistics_descriptive (data,cols):
    num_attributes = data.select_dtypes(include=['int64', 'float64'])
    num_attributes = num_attributes.drop(columns=cols).copy()
    mean_ = pd.DataFrame(num_attributes.apply(np.mean))
    median_ = pd.DataFrame(num_attributes.apply(np.median))
    std_ = pd.DataFrame(num_attributes.apply(np.std))
    max_ = pd.DataFrame(num_attributes.apply(np.max))
    min_ = pd.DataFrame(num_attributes.apply(np.min))
    q1_ = num_attributes.quantile(.25, axis=0)
    q3_ = num_attributes.quantile(.75, axis=0)
    range_ = pd.DataFrame(num_attributes.apply(lambda column: column.max() - column.min()))
    skew_ = pd.DataFrame(num_attributes.apply(lambda column: column.skew()))
    kurtosis_ = pd.DataFrame(num_attributes.apply(lambda column: column.kurtosis()))

    df = pd.concat([mean_, std_, min_, q1_, median_,q3_, max_, range_, skew_, kurtosis_ ], axis=1).reset_index()

    df.columns = ['attributes', 'mean','std', 'min', 'q1', 'median','q3','max', 'range', 'skew', 'kurtosis']

    return df



```


```python
def graphics (data):
    spec = gridspec.GridSpec(3, 4)

    ax0 = plt.subplot(spec[0, 0])
    ax1 = plt.subplot(spec[1, 0])
    ax2 = plt.subplot(spec[2, 0])
    ax3 = plt.subplot(spec[0, 1])
    ax4 = plt.subplot(spec[1, 1])
    ax5 = plt.subplot(spec[2, 1])
    ax6 = plt.subplot(spec[0, 2])
    ax7 = plt.subplot(spec[1, 2])
    ax8 = plt.subplot(spec[2, 2])
    ax9 = plt.subplot(spec[0, 3])
    ax10 = plt.subplot(spec[1, 3])
    ax11 = plt.subplot(spec[2, 3])

    ax = sns.boxplot (data = data, x='price',ax=ax0)
    ax = sns.boxplot (data = data, x='bedrooms',ax=ax1)
    ax = sns.boxplot (data = data, x='bathrooms',ax=ax2)
    ax = sns.boxplot (data = data, x='sqft_living',ax=ax3)
    ax = sns.boxplot (data = data, x='sqft_lot',ax=ax4)
    ax = sns.boxplot (data = data, x='floors', ax=ax5)
    ax = sns.boxplot (data = data, x='sqft_above',ax=ax6)
    ax = sns.boxplot (data = data, x='sqft_basement',ax=ax7)
    ax = sns.boxplot (data = data, x='price/sqft_living',ax=ax8)
    ax = sns.boxplot (data = data, x='price/sqft_lot',ax=ax9)
    
    return None
```


```python
def dict_zipcode(data,col):
    dic = {}
    
    for x in range (len(data)):
        zipcode = data.loc[x,'zipcode']
        price = data.loc[x,col]
        x={zipcode:price}
        dic.update(x)
    
    return dic
```


```python
def recomendation (data,dic,col):
    data['recomendation'] = 'NA'

    for x in range (len(data)):
        zipcode = data.iloc[x,16]

        if ((data.iloc [x,col] < dic[zipcode]) & (data.iloc [x,10] > 3)):
            data.iloc[x,-1] = 'yes'
        else:
            data.iloc[x,-1] = 'no'

    return data
```


```python
#load of the dataframe
path = 'kc_house_data.csv'
data = get_data (path)

#configuration of the float number
pd.set_option('display.float_format', lambda x: '%.2f' % x)
#configuration of image size in screen
sns.set(rc={"figure.figsize":(20, 12)})
#configuration of thetablein screen
pd.set_option('display.max_columns', None)
```


```python
#Sort of the dataframe 'date' to can drop de duplicates 'id'
df = data.sort_values('date', ascending = False).reset_index(drop=True).copy()
df = df.drop_duplicates('id')
#drop of columns data dont have utility in dataset
df = df.drop(['sqft_living15' , 'sqft_lot15'], axis=1)
#create new columns for analises of the dataset
df['price/sqft_living'] = df['price'] / df ['sqft_living']
df['price/sqft_lot'] = df['price'] / df ['sqft_lot']

#filtro de uma linha com quarto outiler pela a analise e filtro das linhas onde o tamanho do lote é igual ou maior que o #tamanho do living
df1 = df[df['bedrooms'] != 33].copy()
df1['l/l'] = df1['sqft_lot'] / df1['sqft_living']
df1 = df1.sort_values('l/l')
df1= df1[df1['l/l']>1].reset_index(drop=True).copy()


#build the table of statistic descriptive
dropcols = ['id', 'waterfront', 'view', 'condition', 'grade', 'yr_built', 'yr_renovated', 'zipcode', 'lat', 'long']
df_sd = statistics_descriptive (df1, dropcols)
#outliner analisys in the first momente we will drop it so later analyses this group case by case
df_sd['iqr'] = (df_sd['q3'] - df_sd['q1'])
df_sd['1.5iqr'] = 1.5 * df_sd['iqr']
df_sd['q1-1.5iqr'] = df_sd['q1'] - df_sd['1.5iqr']
df_sd['q3+1.5iqr'] = df_sd['q3'] + df_sd['1.5iqr']
```


```python
#df1
df_sd
#graphics(df1)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>attributes</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>q1</th>
      <th>median</th>
      <th>q3</th>
      <th>max</th>
      <th>range</th>
      <th>skew</th>
      <th>kurtosis</th>
      <th>iqr</th>
      <th>1.5iqr</th>
      <th>q1-1.5iqr</th>
      <th>q3+1.5iqr</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>price</td>
      <td>543069.33</td>
      <td>371745.74</td>
      <td>75000.00</td>
      <td>320000.00</td>
      <td>450000.00</td>
      <td>650000.00</td>
      <td>7700000.00</td>
      <td>7625000.00</td>
      <td>4.01</td>
      <td>34.20</td>
      <td>330000.00</td>
      <td>495000.00</td>
      <td>-175000.00</td>
      <td>1145000.00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>bedrooms</td>
      <td>3.39</td>
      <td>0.91</td>
      <td>0.00</td>
      <td>3.00</td>
      <td>3.00</td>
      <td>4.00</td>
      <td>11.00</td>
      <td>11.00</td>
      <td>0.49</td>
      <td>1.83</td>
      <td>1.00</td>
      <td>1.50</td>
      <td>1.50</td>
      <td>5.50</td>
    </tr>
    <tr>
      <th>2</th>
      <td>bathrooms</td>
      <td>2.10</td>
      <td>0.77</td>
      <td>0.00</td>
      <td>1.50</td>
      <td>2.25</td>
      <td>2.50</td>
      <td>8.00</td>
      <td>8.00</td>
      <td>0.54</td>
      <td>1.33</td>
      <td>1.00</td>
      <td>1.50</td>
      <td>0.00</td>
      <td>4.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>sqft_living</td>
      <td>2101.64</td>
      <td>925.99</td>
      <td>290.00</td>
      <td>1440.00</td>
      <td>1950.00</td>
      <td>2580.00</td>
      <td>13540.00</td>
      <td>13250.00</td>
      <td>1.44</td>
      <td>5.16</td>
      <td>1140.00</td>
      <td>1710.00</td>
      <td>-270.00</td>
      <td>4290.00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>sqft_lot</td>
      <td>15665.12</td>
      <td>42234.81</td>
      <td>772.00</td>
      <td>5328.25</td>
      <td>7765.50</td>
      <td>10890.00</td>
      <td>1651359.00</td>
      <td>1650587.00</td>
      <td>12.85</td>
      <td>275.23</td>
      <td>5561.75</td>
      <td>8342.62</td>
      <td>-3014.38</td>
      <td>19232.62</td>
    </tr>
    <tr>
      <th>5</th>
      <td>floors</td>
      <td>1.46</td>
      <td>0.51</td>
      <td>1.00</td>
      <td>1.00</td>
      <td>1.00</td>
      <td>2.00</td>
      <td>3.50</td>
      <td>2.50</td>
      <td>0.50</td>
      <td>-0.93</td>
      <td>1.00</td>
      <td>1.50</td>
      <td>-0.50</td>
      <td>3.50</td>
    </tr>
    <tr>
      <th>6</th>
      <td>sqft_above</td>
      <td>1806.72</td>
      <td>836.94</td>
      <td>290.00</td>
      <td>1200.00</td>
      <td>1590.00</td>
      <td>2244.75</td>
      <td>9410.00</td>
      <td>9120.00</td>
      <td>1.41</td>
      <td>3.26</td>
      <td>1044.75</td>
      <td>1567.12</td>
      <td>-367.12</td>
      <td>3811.88</td>
    </tr>
    <tr>
      <th>7</th>
      <td>sqft_basement</td>
      <td>294.91</td>
      <td>447.99</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>580.00</td>
      <td>4820.00</td>
      <td>4820.00</td>
      <td>1.55</td>
      <td>2.59</td>
      <td>580.00</td>
      <td>870.00</td>
      <td>-870.00</td>
      <td>1450.00</td>
    </tr>
    <tr>
      <th>8</th>
      <td>price/sqft_living</td>
      <td>262.68</td>
      <td>110.50</td>
      <td>87.59</td>
      <td>180.85</td>
      <td>242.13</td>
      <td>315.48</td>
      <td>810.14</td>
      <td>722.55</td>
      <td>1.29</td>
      <td>2.21</td>
      <td>134.63</td>
      <td>201.94</td>
      <td>-21.09</td>
      <td>517.41</td>
    </tr>
    <tr>
      <th>9</th>
      <td>price/sqft_lot</td>
      <td>77.22</td>
      <td>66.87</td>
      <td>0.16</td>
      <td>32.74</td>
      <td>57.10</td>
      <td>100.00</td>
      <td>677.63</td>
      <td>677.47</td>
      <td>2.02</td>
      <td>5.61</td>
      <td>67.26</td>
      <td>100.88</td>
      <td>-68.14</td>
      <td>200.88</td>
    </tr>
    <tr>
      <th>10</th>
      <td>l/l</td>
      <td>7.72</td>
      <td>23.45</td>
      <td>1.00</td>
      <td>2.61</td>
      <td>4.17</td>
      <td>6.50</td>
      <td>1640.55</td>
      <td>1639.55</td>
      <td>30.04</td>
      <td>1639.07</td>
      <td>3.89</td>
      <td>5.83</td>
      <td>-3.22</td>
      <td>12.33</td>
    </tr>
  </tbody>
</table>
</div>



Escolhido trabalhar com a mediana ao inves da média para tirar o valor médio por zipcode.


```python
#agrupar por zipcode e tirar a mediana
df.grouped = df1.loc[:,['zipcode','price']].groupby('zipcode').median().reset_index()
#tranformar a tabela em dicionário
dict_zipcode = dict_zipcode(df.grouped,'price')

dict_zipcode_recomendation = recomendation (df1,dict_zipcode,2)
dict_zipcode_recomendation

df2 = dict_zipcode_recomendation[dict_zipcode_recomendation['recomendation'] == 'yes']
df2

df_sd2 = statistics_descriptive (df2, dropcols)
df_sd2

graphics(df2)
```


    
![png](output_11_0.png)
    



```python
groups = []

for group, subset in df1.groupby(by='zipcode'):
    groups2.append({
        'ZipCode': group,
        'Count': len(subset),
        'ID': ','.join((subset.id).astype('str'))
    })

df_recomend = pd.DataFrame(groups2)
df_recomend
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ZipCode</th>
      <th>Count</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>98001</td>
      <td>73</td>
      <td>4463400195,303000445,3353401340,3750605349,321...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>98002</td>
      <td>78</td>
      <td>7116000425,8581400450,3021059155,7116500125,91...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>98003</td>
      <td>55</td>
      <td>2821049082,4222310680,3584000180,2329600040,41...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>98004</td>
      <td>90</td>
      <td>6664000130,3860900111,629000410,4395600060,824...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>98005</td>
      <td>51</td>
      <td>514500090,5307100280,9541800075,2330000035,954...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>65</th>
      <td>98177</td>
      <td>48</td>
      <td>7279300070,1124000050,3626039207,259100110,226...</td>
    </tr>
    <tr>
      <th>66</th>
      <td>98178</td>
      <td>26</td>
      <td>185000118,1180008315,3348401622,8068000585,427...</td>
    </tr>
    <tr>
      <th>67</th>
      <td>98188</td>
      <td>17</td>
      <td>7148200040,5379805460,1003600080,4435000490,10...</td>
    </tr>
    <tr>
      <th>68</th>
      <td>98198</td>
      <td>56</td>
      <td>1697000370,6052401215,3530410020,7686203275,95...</td>
    </tr>
    <tr>
      <th>69</th>
      <td>98199</td>
      <td>51</td>
      <td>6821102100,6821100090,6822100030,8127700210,13...</td>
    </tr>
  </tbody>
</table>
<p>70 rows × 3 columns</p>
</div>




```python
df3_2 = dict_zipcode_recomendation2[(dict_zipcode_recomendation2['recomendation'] == 'yes')].sort_values(['condition','price'], ascending =[False,True]).copy()
#graphics(df3_2)

```


```python
df3_2[df3_2['id'] == 303000445]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>date</th>
      <th>price</th>
      <th>bedrooms</th>
      <th>bathrooms</th>
      <th>sqft_living</th>
      <th>sqft_lot</th>
      <th>floors</th>
      <th>waterfront</th>
      <th>view</th>
      <th>condition</th>
      <th>grade</th>
      <th>sqft_above</th>
      <th>sqft_basement</th>
      <th>yr_built</th>
      <th>yr_renovated</th>
      <th>zipcode</th>
      <th>lat</th>
      <th>long</th>
      <th>price/sqft_living</th>
      <th>price/sqft_lot</th>
      <th>l/l</th>
      <th>recomendation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>20073</th>
      <td>303000445</td>
      <td>20140523T000000</td>
      <td>175000.00</td>
      <td>2</td>
      <td>1.00</td>
      <td>1300</td>
      <td>44431</td>
      <td>1.00</td>
      <td>0</td>
      <td>0</td>
      <td>5</td>
      <td>6</td>
      <td>1300</td>
      <td>0</td>
      <td>1958</td>
      <td>0</td>
      <td>98001</td>
      <td>47.33</td>
      <td>-122.27</td>
      <td>134.62</td>
      <td>3.94</td>
      <td>34.18</td>
      <td>yes</td>
    </tr>
  </tbody>
</table>
</div>




```python
dataprice_price_sqft_living = data_zipcode [['price/sqft_living','zipcode']].groupby('zipcode').mean().reset_index()
dict_zipcode_sqft_living = dict_zipcode(dataprice_price_sqft_living,'price/sqft_living')

```


```python


```


```python

```


```python
df1 = df[(df['price'] <= 1605402) & 
      (df['bedrooms'] <= 7) &
      (df['bathrooms'] <= 4.75) &
      (df['sqft_living'] <= 5910) &
      (df['sqft_lot'] <= 27665) &
      (df['floors'] <= 5) &
      (df['sqft_above'] <= 5280) &
      (df['sqft_basement'] <= 2240) &
      (df['price/sqft_living'] <= 726.09) &
      (df['price/sqft_lot'   ] <= 330.93)] [['id', 'date', 'price', 'bedrooms', 'bathrooms', 'sqft_living', 'sqft_lot',
                                           'floors',   'waterfront', 'view', 'condition', 'grade','sqft_above', 
                                           'sqft_basement', 'yr_built', 'yr_renovated', 'zipcode','lat', 'long', 
                                           'price/sqft_living', 'price/sqft_lot']]
```


```python

```


```python

```


```python

```


```python

```
