```python
import matplotlib.pyplot as plt
import wordcloud
import pandas as pd
from IPython.display import display
```


```python
places_df = pd.read_csv('../data/places_GP.csv')
df = pd.read_csv('../data/comments_filled_sent.csv')
words = pd.read_csv('../data/words.csv')
bigramms = pd.read_csv('../data/bigramms.csv')
```

    C:\Users\yupes\anaconda3\envs\python-gis\lib\site-packages\IPython\core\interactiveshell.py:3444: DtypeWarning: Columns (1,4) have mixed types.Specify dtype option on import or set low_memory=False.
      exec(code_obj, self.user_global_ns, self.user_ns)
    


```python
comments_df = pd.read_csv('../data/comments.csv')
```

    C:\Users\yupes\anaconda3\envs\python-gis\lib\site-packages\IPython\core\interactiveshell.py:3444: DtypeWarning: Columns (1,4) have mixed types.Specify dtype option on import or set low_memory=False.
      exec(code_obj, self.user_global_ns, self.user_ns)
    


```python
comments_df = comments_df.drop_duplicates()
```


```python
comments_df.shape
```




    (414137, 10)




```python
places_df_sent = places_df.merge(comments_df.groupby('url').count()[['reviewer_page']].reset_index().rename(columns = {'reviewer_page':'count_downloaded'}).merge(df.groupby('url').count().sort_values('index_review')[['index_review']].reset_index().rename(columns = {'index_review':'count_filled'}).merge(
df.loc[df.score>0].groupby('url').count().sort_values('index_review')[['index_review']].reset_index().rename(columns = {'index_review':'count_pos'}), \
how = 'left').merge(df.loc[df.score<0].groupby('url').count().sort_values('index_review')[['index_review']].reset_index().rename(columns = {'index_review':'count_neg'}), how = 'left')
, how = 'left', on = 'url'), how = 'left', on = 'url').copy()
```


```python
places_df_sent.loc[places_df_sent.count_downloaded==0, 'count_downloaded'] = places_df_sent.loc[places_df_sent.count_downloaded==0, 'count_review']
```


```python
places_df_sent = places_df_sent.merge(df.groupby('url').median()[['score']].rename(columns = {'score':'median_score'}).reset_index()\
    .merge(df.loc[df.score<0].groupby('url').median()[['score']].rename(columns = {'score':'median_negative'}).reset_index(), how = 'left', on = 'url')\
    .merge(df.loc[df.score>0].groupby('url').median()[['score']].rename(columns = {'score':'median_positive'}).reset_index(), how = 'left', on ='url'), how = 'left', on = 'url')
```


```python
for i in ['count_review', 'count_downloaded', 'count_filled','count_pos', 'count_neg']:
    places_df_sent[i].fillna(0, inplace = True)
```


```python
places_df_sent['count_neutral'] = places_df_sent['count_filled']-places_df_sent['count_pos']-places_df_sent['count_neg']
```


```python
places_df_sent.to_csv('../data/places_GP_sent.csv', index = False)
```


```python
# функция для рисования облака слов
def get_cloud(words_list, title, cm, path):
    stop = ["который", "этот_магазин","вообще", "которое","которые","много","сказал", "ничего","второй_раз","этот_гостиница","этот_церковь", "этот_автобус", "один_место","этот_храм","свой_дело", "такой_ощущение", "этот_парк", "один_сторона", "свой_время", "один_раз", "первый_раз", "этот_отель","данный_магазин", "другой_магазин","какой", "такой", "тот", "если","только", "есть","была", "место", "месте", "было", "были", "тогда","очень", "этот", "другой", "также", "чтобы", "быть", "мочь", "свой", "весь", "один"]

    wc = wordcloud.WordCloud(collocations=True, 
                             background_color = 'white', 
                             width=1600, 
                             height=800, 
                             min_word_length=4, 
                             colormap=cm,
                             stopwords= stop).generate(" ".join(words_list))
    
    plt.figure(figsize=(20,10))
    plt.imshow(wc, interpolation="bilinear")
    plt.title(title)
    plt.axis("off")
    plt.show()
    wc.to_file(path+'/'+title+'.png')
```

# Анализ слов


```python
g = ['NOUN','ADJF', 'ADJS', 'COMP','VERB', 'INFN', 'PRTF', 'PRTS', 'GRND', 'ADVB']
```


```python
words.word.replace('магазине', 'магазин', inplace = True)
words.word.replace('парка', 'парк', inplace = True)
words.word.replace('парке', 'парк', inplace = True)
words.word.replace('моста', 'мост', inplace = True)
words.word.replace('мосту', 'мост', inplace = True)
words.word.replace('детям', 'дети', inplace = True)
words.word.replace('деревьев', 'деревья', inplace = True)
words.word.replace('детьми', 'дети', inplace = True)
words.word.replace('детей', 'дети', inplace = True)
words.word.replace('животных', 'животные', inplace = True)
words.word.replace('клуба', 'клуб', inplace = True)
words.word.replace('номера', 'номер', inplace = True)
words.word.replace('номере', 'номер', inplace = True)
words.word.replace('реки', 'река', inplace = True)
words.word.replace('реку', 'река', inplace = True)
words.word.replace('аттракционов', 'аттракционы', inplace = True)
words.word.replace('врачи', 'врач', inplace = True)
words.word.replace('парка', 'парк', inplace = True)
words.word.replace('краснодара', 'краснодар', inplace = True)
words.word.replace('краснодаре', 'краснодар', inplace = True)
words.word.replace('прогулкок', 'прогулки', inplace = True)
words.word.replace('деревьями', 'деревья', inplace = True)
```


```python
for i in words.loc[~words.category.isna()].category.drop_duplicates()[:-1]:
#     display(words.loc[((words.category==i) & (words.grammema.isin(g)) & (words.score<0)) , 'word'].value_counts()[:5])
    
    get_cloud(words.loc[((words.category==i) & (words.grammema.isin(g)) & (words.score<0)) , 'word'].to_list(), i+' (негативные)', 'PuBu', '../data/charts/words')
    get_cloud(words.loc[((words.category==i) & (words.grammema.isin(g)) & (words.score>0)) , 'word'].to_list(), i+' (позитивные)', 'OrRd','../data/charts/words')
```


    
![png](output_15_0.png)
    



    
![png](output_15_1.png)
    



    
![png](output_15_2.png)
    



    
![png](output_15_3.png)
    



    
![png](output_15_4.png)
    



    
![png](output_15_5.png)
    



    
![png](output_15_6.png)
    



    
![png](output_15_7.png)
    



    
![png](output_15_8.png)
    



    
![png](output_15_9.png)
    



    
![png](output_15_10.png)
    



    
![png](output_15_11.png)
    



    
![png](output_15_12.png)
    



    
![png](output_15_13.png)
    



    
![png](output_15_14.png)
    



    
![png](output_15_15.png)
    



    
![png](output_15_16.png)
    



    
![png](output_15_17.png)
    



    
![png](output_15_18.png)
    



    
![png](output_15_19.png)
    



    
![png](output_15_20.png)
    



    
![png](output_15_21.png)
    



    
![png](output_15_22.png)
    



    
![png](output_15_23.png)
    



    
![png](output_15_24.png)
    



    
![png](output_15_25.png)
    



    
![png](output_15_26.png)
    



    
![png](output_15_27.png)
    



    
![png](output_15_28.png)
    



    
![png](output_15_29.png)
    



    
![png](output_15_30.png)
    



    
![png](output_15_31.png)
    



    
![png](output_15_32.png)
    



    
![png](output_15_33.png)
    


# Анализ биграмм


```python
bigramms
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
      <th>index_review</th>
      <th>bigramm</th>
      <th>word1</th>
      <th>word2</th>
      <th>tag1</th>
      <th>tag2</th>
      <th>url</th>
      <th>score</th>
      <th>positive</th>
      <th>negative</th>
      <th>comparative</th>
      <th>category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>207236</td>
      <td>('отличный', 'магазин')</td>
      <td>отличный</td>
      <td>магазин</td>
      <td>ADJF</td>
      <td>NOUN</td>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>8.3</td>
      <td>8.3</td>
      <td>0.0</td>
      <td>2.075000</td>
      <td>Магазин</td>
    </tr>
    <tr>
      <th>1</th>
      <td>207237</td>
      <td>('ужасный', 'обслуживание')</td>
      <td>ужасный</td>
      <td>обслуживание</td>
      <td>ADJF</td>
      <td>NOUN</td>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>-5.0</td>
      <td>5.0</td>
      <td>-10.0</td>
      <td>-0.098039</td>
      <td>Магазин</td>
    </tr>
    <tr>
      <th>2</th>
      <td>207237</td>
      <td>('обслуживание', 'магазин')</td>
      <td>обслуживание</td>
      <td>магазин</td>
      <td>NOUN</td>
      <td>NOUN</td>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>-5.0</td>
      <td>5.0</td>
      <td>-10.0</td>
      <td>-0.098039</td>
      <td>Магазин</td>
    </tr>
    <tr>
      <th>3</th>
      <td>207237</td>
      <td>('магазин', 'продавец')</td>
      <td>магазин</td>
      <td>продавец</td>
      <td>NOUN</td>
      <td>NOUN</td>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>-5.0</td>
      <td>5.0</td>
      <td>-10.0</td>
      <td>-0.098039</td>
      <td>Магазин</td>
    </tr>
    <tr>
      <th>4</th>
      <td>207237</td>
      <td>('продавец', 'вообще')</td>
      <td>продавец</td>
      <td>вообще</td>
      <td>NOUN</td>
      <td>ADVB</td>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>-5.0</td>
      <td>5.0</td>
      <td>-10.0</td>
      <td>-0.098039</td>
      <td>Магазин</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1279680</th>
      <td>336213</td>
      <td>('есть', 'выбор')</td>
      <td>есть</td>
      <td>выбор</td>
      <td>INFN</td>
      <td>NOUN</td>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>5.0</td>
      <td>5.0</td>
      <td>0.0</td>
      <td>1.250000</td>
      <td>Магазин</td>
    </tr>
    <tr>
      <th>1279681</th>
      <td>336214</td>
      <td>('большой', 'выбор')</td>
      <td>большой</td>
      <td>выбор</td>
      <td>ADJF</td>
      <td>NOUN</td>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>5.0</td>
      <td>5.0</td>
      <td>0.0</td>
      <td>0.714286</td>
      <td>Магазин</td>
    </tr>
    <tr>
      <th>1279682</th>
      <td>336214</td>
      <td>('выбор', 'качественный')</td>
      <td>выбор</td>
      <td>качественный</td>
      <td>NOUN</td>
      <td>ADJF</td>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>5.0</td>
      <td>5.0</td>
      <td>0.0</td>
      <td>0.714286</td>
      <td>Магазин</td>
    </tr>
    <tr>
      <th>1279683</th>
      <td>336214</td>
      <td>('качественный', 'обувь')</td>
      <td>качественный</td>
      <td>обувь</td>
      <td>ADJF</td>
      <td>NOUN</td>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>5.0</td>
      <td>5.0</td>
      <td>0.0</td>
      <td>0.714286</td>
      <td>Магазин</td>
    </tr>
    <tr>
      <th>1279684</th>
      <td>336214</td>
      <td>('весь', 'семья')</td>
      <td>весь</td>
      <td>семья</td>
      <td>ADJF</td>
      <td>NOUN</td>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>5.0</td>
      <td>5.0</td>
      <td>0.0</td>
      <td>0.714286</td>
      <td>Магазин</td>
    </tr>
  </tbody>
</table>
<p>1279685 rows × 12 columns</p>
</div>




```python
for i in bigramms.loc[~bigramms.category.isna()].category.drop_duplicates():
    try:
        get_cloud(bigramms.loc[((bigramms.category==i) & (bigramms.tag1.isin(['ADJF', 'ADJS']))& (bigramms.tag2 == 'NOUN') & (bigramms.score<0)) , 'bigramm'].str[2:-2].str.split("', '").str.join('_').to_list(), i+' (негативные)', 'PuBu','../data/charts/bigramms')
        get_cloud(bigramms.loc[((bigramms.category==i) & (bigramms.tag1.isin(['ADJF', 'ADJS']))& (bigramms.tag2 == 'NOUN') & (bigramms.score>0)) , 'bigramm'].str[2:-2].str.split("', '").str.join('_').to_list(), i+' (позитивные)', 'OrRd','../data/charts/bigramms')
    
    except:
        print(i)
```


    
![png](output_18_0.png)
    



    
![png](output_18_1.png)
    



    
![png](output_18_2.png)
    



    
![png](output_18_3.png)
    



    
![png](output_18_4.png)
    



    
![png](output_18_5.png)
    



    
![png](output_18_6.png)
    



    
![png](output_18_7.png)
    



    
![png](output_18_8.png)
    



    
![png](output_18_9.png)
    



    
![png](output_18_10.png)
    



    
![png](output_18_11.png)
    



    
![png](output_18_12.png)
    



    
![png](output_18_13.png)
    



    
![png](output_18_14.png)
    



    
![png](output_18_15.png)
    



    
![png](output_18_16.png)
    



    
![png](output_18_17.png)
    



    
![png](output_18_18.png)
    



    
![png](output_18_19.png)
    



    
![png](output_18_20.png)
    



    
![png](output_18_21.png)
    



    
![png](output_18_22.png)
    



    
![png](output_18_23.png)
    



    
![png](output_18_24.png)
    



    
![png](output_18_25.png)
    



    
![png](output_18_26.png)
    



    
![png](output_18_27.png)
    



    
![png](output_18_28.png)
    



    
![png](output_18_29.png)
    



    
![png](output_18_30.png)
    



    
![png](output_18_31.png)
    



    
![png](output_18_32.png)
    



    
![png](output_18_33.png)
    


    Локальность
    


```python
df = df.merge(places_df[['url', 'category']], how = 'left')
```

# Анализ сентиментов


```python
print('Распределение среднего числа отзывов:')
display(places_df_sent.loc[places_df_sent.count_filled>0][['count_pos', 'count_neutral','count_neg']].mean())
```

    Распределение среднего числа отзывов:
    


    count_pos        20.262791
    count_neutral     5.377061
    count_neg         2.621376
    dtype: float64



```python
places_df_sent.loc[places_df_sent.count_filled>0][['count_pos', 'count_neutral','count_neg']].mean().plot(kind = 'pie', colors =[ "#E07A5F", "#9c9c9c","#3D405B"], labels = ['Позитивные', "Нейтральные", "Негативные"], figsize = (16, 8), autopct='%1.1f%%')
plt.ylabel('')
plt.title('Распределение количества отзывов по сентиментам Аттракторов')
```




    Text(0.5, 1.0, 'Распределение количества отзывов по сентиментам Аттракторов')




    
![png](output_22_1.png)
    



```python
plot_df = df.loc[df.score>0].groupby('category').median()[['score']].rename(columns = {'score':'positive'}).reset_index().sort_values('positive', ascending = False).merge(
          df.loc[df.score<0].groupby('category').median()[['score']].rename(columns = {'score':'negative'}).reset_index(), how = 'left', on = 'category'
)
plt.figure(figsize = (16, 8))
ax = plt.subplot(111)
ax.bar(plot_df['category'],plot_df['positive'], color =["#E07A5F"])
ax.bar(plot_df['category'],plot_df['negative'], color =["#3D405B"])
plt.gcf().autofmt_xdate()
plt.xlabel('Категории')
plt.ylabel('Медиана сентимента отзывов')
plt.title('Распределение категорий по медиане сентимента отзывов')
plt.legend(['позитивные', "негативные"])
```




    <matplotlib.legend.Legend at 0x1aa87a8a9d0>




    
![png](output_23_1.png)
    



```python
df.groupby('category').mean()[['score']].sort_values('score', ascending = False).plot.bar(color = ["#F2CC8F"],figsize = (16, 8))
plt.gcf().autofmt_xdate()
plt.xlabel('Категории')
plt.legend(['оценка сентимента'])
plt.ylabel('Средний сентимент отзывов')
plt.title('Распределение категорий по средней оценке сентимента отзывов')
```




    Text(0.5, 1.0, 'Распределение категорий по средней оценке сентимента отзывов')




    
![png](output_24_1.png)
    



```python
places_df_sent.groupby('category').median()[['count_pos', 'count_neg']].sort_values('count_pos', ascending = False)\
    .plot.bar(color =["#E07A5F", "#3D405B"],figsize = (16, 8))
plt.gcf().autofmt_xdate()
plt.xlabel('Категории')
plt.legend(['позитивные', "негативные"])
plt.ylabel('Количество отзывов')
plt.title('Распределение отзывов по сентиментам')
```




    Text(0.5, 1.0, 'Распределение отзывов по сентиментам')




    
![png](output_25_1.png)
    


# Аттракторы


```python
indexs = [1609,1616, 15198,15267,15248,15196,252,15233,15297,15207]
```


```python
places_df_sent.loc[places_df_sent.index.isin(indexs)]
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
      <th>url</th>
      <th>title</th>
      <th>address</th>
      <th>type</th>
      <th>popular_times_bars</th>
      <th>geometry</th>
      <th>types</th>
      <th>rating</th>
      <th>count_review</th>
      <th>count_photo</th>
      <th>...</th>
      <th>normalized_rating</th>
      <th>rating_by_category</th>
      <th>count_downloaded</th>
      <th>count_filled</th>
      <th>count_pos</th>
      <th>count_neg</th>
      <th>count_neutral</th>
      <th>median_score</th>
      <th>median_negative</th>
      <th>median_positive</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>252</th>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>Городской сад, парк культуры и отдыха</td>
      <td>ул. Постовая, 34, Краснодар, Краснодарский кра...</td>
      <td>Парк</td>
      <td>['Загруженность в 06:00: 0%.', 'Загруженность ...</td>
      <td>POINT (38.9699221 45.0105502)</td>
      <td>['park', 'tourist_attraction', 'point_of_inter...</td>
      <td>4.6</td>
      <td>6826.0</td>
      <td>5125.0</td>
      <td>...</td>
      <td>1.195743</td>
      <td>1.195743</td>
      <td>929.0</td>
      <td>929.0</td>
      <td>773.0</td>
      <td>67.0</td>
      <td>89.0</td>
      <td>6.7</td>
      <td>-5.0</td>
      <td>7.5</td>
    </tr>
    <tr>
      <th>1609</th>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>Парк «Краснодар»</td>
      <td>Краснодар, Краснодарский край, 350059</td>
      <td>Парк</td>
      <td>['Загруженность в 04:00: 0%.', 'Загруженность ...</td>
      <td>POINT (39.0317799 45.0416363)</td>
      <td>['park', 'tourist_attraction', 'point_of_inter...</td>
      <td>4.9</td>
      <td>35831.0</td>
      <td>60126.0</td>
      <td>...</td>
      <td>2.980000</td>
      <td>2.980000</td>
      <td>930.0</td>
      <td>930.0</td>
      <td>897.0</td>
      <td>9.0</td>
      <td>24.0</td>
      <td>12.5</td>
      <td>-5.0</td>
      <td>12.5</td>
    </tr>
    <tr>
      <th>1616</th>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>Стадион ФК "Краснодар"</td>
      <td>ул. Разведчика Леонова, 1, Краснодар, Краснода...</td>
      <td>Стадион</td>
      <td>NaN</td>
      <td>POINT (39.0291908 45.0444895)</td>
      <td>['stadium', 'gym', 'health', 'point_of_interes...</td>
      <td>4.9</td>
      <td>25865.0</td>
      <td>33754.0</td>
      <td>...</td>
      <td>2.263249</td>
      <td>2.980000</td>
      <td>930.0</td>
      <td>930.0</td>
      <td>869.0</td>
      <td>21.0</td>
      <td>40.0</td>
      <td>10.0</td>
      <td>-5.0</td>
      <td>10.0</td>
    </tr>
    <tr>
      <th>15196</th>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>Ботанический сад им. И.С. Косенко (Дендрарий)</td>
      <td>ул. Калинина, 13, Краснодар, Краснодарский кра...</td>
      <td>Ботанический сад</td>
      <td>['Загруженность в 05:00: 0%.', 'Загруженность ...</td>
      <td>POINT (38.9293339 45.0541482)</td>
      <td>['tourist_attraction', 'park', 'point_of_inter...</td>
      <td>4.8</td>
      <td>5265.0</td>
      <td>6161.0</td>
      <td>...</td>
      <td>1.209408</td>
      <td>1.209408</td>
      <td>930.0</td>
      <td>930.0</td>
      <td>827.0</td>
      <td>34.0</td>
      <td>69.0</td>
      <td>7.5</td>
      <td>-4.2</td>
      <td>8.3</td>
    </tr>
    <tr>
      <th>15198</th>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>Чистяковская Роща</td>
      <td>ул. Колхозная, Краснодар, Краснодарский край, ...</td>
      <td>Парк</td>
      <td>['Загруженность в 04:00: 1%.', 'Загруженность ...</td>
      <td>POINT (38.9940405 45.05764529999999)</td>
      <td>['park', 'tourist_attraction', 'point_of_inter...</td>
      <td>4.6</td>
      <td>20277.0</td>
      <td>18542.0</td>
      <td>...</td>
      <td>1.794292</td>
      <td>1.794292</td>
      <td>930.0</td>
      <td>930.0</td>
      <td>804.0</td>
      <td>47.0</td>
      <td>79.0</td>
      <td>6.7</td>
      <td>-3.3</td>
      <td>7.5</td>
    </tr>
    <tr>
      <th>15207</th>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>Сквер им. Г.К. Жукова</td>
      <td>ул. Красная, Краснодар, Краснодарский край, 35...</td>
      <td>Парк</td>
      <td>['Загруженность в 04:00: 1%.', 'Загруженность ...</td>
      <td>POINT (38.9711376 45.0242461)</td>
      <td>['park', 'tourist_attraction', 'point_of_inter...</td>
      <td>4.7</td>
      <td>6553.0</td>
      <td>10395.0</td>
      <td>...</td>
      <td>1.295773</td>
      <td>1.295773</td>
      <td>929.0</td>
      <td>929.0</td>
      <td>812.0</td>
      <td>28.0</td>
      <td>89.0</td>
      <td>5.8</td>
      <td>-4.1</td>
      <td>7.5</td>
    </tr>
    <tr>
      <th>15233</th>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>Мост Поцелуев</td>
      <td>ул. Кубанонабережная, Краснодар, Краснодарский...</td>
      <td>Мост</td>
      <td>['Загруженность в 04:00: 1%.', 'Загруженность ...</td>
      <td>POINT (38.9577894 45.0253057)</td>
      <td>['tourist_attraction', 'point_of_interest', 'e...</td>
      <td>4.4</td>
      <td>6223.0</td>
      <td>4233.0</td>
      <td>...</td>
      <td>1.124079</td>
      <td>2.956522</td>
      <td>930.0</td>
      <td>930.0</td>
      <td>673.0</td>
      <td>102.0</td>
      <td>155.0</td>
      <td>5.0</td>
      <td>-5.0</td>
      <td>7.5</td>
    </tr>
    <tr>
      <th>15248</th>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>Екатерининский сквер</td>
      <td>Краснодар, Краснодарский край, 350063</td>
      <td>Парк</td>
      <td>['Загруженность в 04:00: 1%.', 'Загруженность ...</td>
      <td>POINT (38.9686925 45.0156085)</td>
      <td>['park', 'tourist_attraction', 'point_of_inter...</td>
      <td>4.8</td>
      <td>4028.0</td>
      <td>12028.0</td>
      <td>...</td>
      <td>1.272463</td>
      <td>1.272463</td>
      <td>930.0</td>
      <td>930.0</td>
      <td>821.0</td>
      <td>20.0</td>
      <td>89.0</td>
      <td>5.8</td>
      <td>-5.0</td>
      <td>7.5</td>
    </tr>
    <tr>
      <th>15267</th>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>Парк культуры и отдыха имени 30-летия Победы</td>
      <td>ул. Береговая, 146, Краснодар, Краснодарский к...</td>
      <td>Парк</td>
      <td>['Загруженность в 04:00: 0%.', 'Загруженность ...</td>
      <td>POINT (38.95178420000001 45.0169693)</td>
      <td>['park', 'tourist_attraction', 'point_of_inter...</td>
      <td>4.6</td>
      <td>8236.0</td>
      <td>18525.0</td>
      <td>...</td>
      <td>1.457960</td>
      <td>1.457960</td>
      <td>930.0</td>
      <td>930.0</td>
      <td>776.0</td>
      <td>53.0</td>
      <td>101.0</td>
      <td>6.7</td>
      <td>-5.0</td>
      <td>7.5</td>
    </tr>
    <tr>
      <th>15297</th>
      <td>https://www.google.com/maps/place/?q=place_id:...</td>
      <td>ЦПКиО им. Горького</td>
      <td>ул. Постовая, 34 строение 2, Краснодар, Красно...</td>
      <td>Парк</td>
      <td>['Загруженность в 06:00: 0%.', 'Загруженность ...</td>
      <td>POINT (38.970636 45.0120846)</td>
      <td>['park', 'point_of_interest', 'establishment']</td>
      <td>4.6</td>
      <td>4542.0</td>
      <td>12069.0</td>
      <td>...</td>
      <td>1.247490</td>
      <td>1.247490</td>
      <td>930.0</td>
      <td>930.0</td>
      <td>796.0</td>
      <td>49.0</td>
      <td>85.0</td>
      <td>5.8</td>
      <td>-3.3</td>
      <td>7.5</td>
    </tr>
  </tbody>
</table>
<p>10 rows × 22 columns</p>
</div>




```python
places_df_sent.loc[places_df_sent.index.isin(indexs)].to_csv('../data/places_GP_attractors.csv', index = False)
```


```python
places_df_sent.loc[places_df_sent.index.isin(indexs)].set_index('title')[['count_pos', 'count_neg']].sort_values('count_pos', ascending = False)\
    .plot.bar(color =["#3D405B", "#E07A5F"],figsize = (16, 8))
plt.gcf().autofmt_xdate()
plt.xlabel('Аттракторы')
plt.ylabel('Количество отзывов')

plt.legend(['позитивные', "негативные"])
plt.title('Распределение отзывов по сентиментам')
```




    Text(0.5, 1.0, 'Распределение отзывов по сентиментам')




    
![png](output_30_1.png)
    



```python
places_df_sent.columns
```




    Index(['url', 'title', 'address', 'type', 'popular_times_bars', 'geometry',
           'types', 'rating', 'count_review', 'count_photo', 'category_2',
           'category', 'normalized_rating', 'rating_by_category',
           'count_downloaded', 'count_filled', 'count_pos', 'count_neg',
           'count_neutral', 'median_score', 'median_negative', 'median_positive'],
          dtype='object')




```python
plot_df = places_df_sent.loc[places_df_sent.index.isin(indexs)][['title','median_positive', 'median_negative']].sort_values('median_positive', ascending = False)

plt.figure(figsize = (16, 8))
ax = plt.subplot(111)
ax.bar(plot_df['title'],plot_df['median_positive'], color =["#E07A5F"])
ax.bar(plot_df['title'],plot_df['median_negative'], color =["#3D405B"])
plt.gcf().autofmt_xdate()
plt.xlabel('Аттракторы')
plt.ylabel('Медиана отзывов')
plt.legend(['позитивные', "негативные"])
plt.title('Распределение отзывов по сентиментам')
```




    Text(0.5, 1.0, 'Распределение отзывов по сентиментам')




    
![png](output_32_1.png)
    



```python
places_df_sent.loc[places_df_sent.index.isin(indexs)][['count_pos', 'count_neutral','count_neg']].median().plot(kind = 'pie', colors =[ "#E07A5F", "#9c9c9c","#3D405B"], labels = ['Позитивные', "Нейтральные", "Негативные"], figsize = (16, 8), autopct='%1.1f%%')
plt.ylabel('')
plt.title('Распределение количества отзывов по сентиментам Аттракторов')
```




    Text(0.5, 1.0, 'Распределение количества отзывов по сентиментам Аттракторов')




    
![png](output_33_1.png)
    



```python
attractors_w = words.loc[words.url.isin(places_df_sent.loc[places_df_sent.index.isin(indexs), 'url'])]
get_cloud(attractors_w.loc[((attractors_w.grammema == 'NOUN') & (attractors_w.score<0)) , 'word'].to_list(), 'Сущности по аттракторам (негативные)', 'PuBu')
get_cloud(attractors_w.loc[((attractors_w.grammema == 'NOUN') & (attractors_w.score>0)) , 'word'].to_list(), 'Сущности по аттракторам (позитивные)', 'OrRd')
```


    
![png](output_34_0.png)
    



    
![png](output_34_1.png)
    



```python
attractors_b = bigramms.loc[bigramms.url.isin(places_df_sent.loc[places_df_sent.index.isin(indexs), 'url'])]
get_cloud(attractors_b.loc[((attractors_b.tag1.isin(['ADJF', 'ADJS']))& (attractors_b.tag2 == 'NOUN') & (attractors_b.score<0)) , 'bigramm'].str[2:-2].str.split("', '").str.join('_').to_list(), 'Биграммы по аттракторам (негативные)', 'PuBu')
get_cloud(attractors_b.loc[((attractors_b.tag1.isin(['ADJF', 'ADJS']))& (attractors_b.tag2 == 'NOUN') & (attractors_b.score>0)) , 'bigramm'].str[2:-2].str.split("', '").str.join('_').to_list(), 'Биграммы по аттракторам (позитивные)', 'OrRd')
```


    
![png](output_35_0.png)
    



    
![png](output_35_1.png)
    

