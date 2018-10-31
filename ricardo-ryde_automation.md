
# Takeaways from quick exploration in Excel

After quickly analyzing the file in Excel, these would be my first recommendations to management:

1. Take superman off the streets. He's responsible for 23 of the 30 near-misses in just one day. He also has the lowest rating. If needed, use this quote:

  > Today is a day for truth. The world needs to know what happened and to know what he stands for. That kind of power is very dangerous.

1. End long rides. They cost the same as short ones, but leave riders very unsatisfied. They don't like it. Most importantly, that's were near misses happen.

1. Promote Hulk. Or if you had to pick just one model to mass produce, just pick Hulk. Zero near-misses, highest rating, most trips, most riders, and most revenue (by far! He brought in \\$514, leaving the second highest, robin, at \\$113 away with just above \\$400).

    _(Uhm, Hulk is the best, and Superman is the worst. Sounds like we have an Avengers fan in the midst.)_

1. Promote multiple rider "pools". We charge a \\$3 premium for 1-passenger (private) rides and that doesn't seem to discourage people from taking them or having a positive experience. And they're more lucrative than 2-rider pools. But on Oct 2nd we brought in a total of \\$1114 for almost 200 private rides vs. \\$8372 for more than 800 pools. So not only pools seem more popular, but they also bring twice as much money as private ones. The 4-rider are both particularly lucrative and have the highest rating.

1. Consider limiting number of riders to 4. 5-rider trips have much lower ratings (car is packed?)

1. Disregard all of the above. These are all based on just **one day** of data. Please don't make any rash decisions.

Other learnings:

1. Users still return after near misses. Most striking is user #7 that continued to ride, even after suffering 5 near misses in one trip with superman. The user even rode superman again. He probably didn't have a choice. The same is true for other users.

1. The trips are divided in two buckets of duration: less than 30 min (aka _short trips_), and longer than one hour (_long trips_). All the near misses happen in a long trip. That's 30 near misses in 51 trips. Some trips have multiple near misses (the maximum being 5 in just one trip, by superman), and so there's 19 trips with at least one near miss. These are very high rates.

1. There are multiple instances of the same car. Unless the laws of physics are being violated (or the timestamps are wrong), we must assume there is more than one superman. For example, there's a superman trip from `2018-10-02 07:12:21` to `2018-10-02 08:21:23`, and another from `2018-10-02 07:13:45` to `2018-10-02 08:32:29`. These are overlapping. More importantly, near misses happen in both these trips. So the problem is not with just one particular instance of superman, but the whole model. This happens for other cars as well.

1. Prices are also divided in two buckets: \\$5 to \\$6.5 for 1 rider, and \\$2 to \\$3.5 for multiple riders (2 to 5 riders, it doesn't matter). Prices also don't seem to depend on distance. That means that total revenue is just a function of number of rides (with the exception of the 1-rider vs multiple riders price difference). We could change our pricing model to depend on distance and/or time, but riders don't seem to like long-distance rides anyway. Higher price would just make it worse!

1. All cars take 1 to 5 passengers. Seems to suggest they're all the same size.

1. There is an error in the timestamps. Later trips seem to travel back in time because they end in the same day, but much earlier. E.g. from `2018-10-02 23:59:36` to `2018-10-02 00:10:15`. I just assume the end date was meant to be `2018-10-03 00:10:15` instead (i.e. one day later). That's easy to fix by adding 1 day to the trip duration if the duration is negative.


# Using python and sklearn for predictive modeling
I usually do data exploration with python, pandas, and seaborn (for plotting). And because you probably want to evaluate my python skills, I'll do a bit of exploration here. But in the end I don't think we learned anything new that we didn't already know after exploring a bit using Excel and pivot tables.

Towards the bottom of this notebook we do a tiny bit of modeling. We'll use predictive models, but the goal is not to predict or forecast. The goal is to explore the importance of some of these features. For example, we've seen that the car matters, so does the trip duration. We analyzed most features independently, but here we're looking for more nuanced relationships, that we might have missed at first glance.

Finally, we should note that the sample size is very small. We add more features, but mostly are just transforms of existing ones, highly correlated. We expand the feature space a little bit, but not significantly. Again, we're not doing predictive modeling, otherwise we would worry about having too many (collinear) features. We also don't need to split the data into test and training for feature importance. Basically we're throwing out all precautions usually in place to avoid overfitting out the window just for this purpose (i.e. low sample size, feature collinearity, multi-dimensionality, no cross-validation, and class imbalance – near misses are a "rare" event). We could do more model if we had the time, but again, probably not worth it without more data.


```python
import datetime
import numpy as np
import pandas as pd
```


```python
df = pd.read_excel('final_analytics_takehome (1) (1) (1).xlsx')
print (df.shape)
print (df.dtypes)
df.head()
```

    (1033, 9)
    user_id                     int64
    car_id                     object
    start_time         datetime64[ns]
    end_time           datetime64[ns]
    num_riders                  int64
    region                     object
    num_near_misses             int64
    price                     float64
    rating                      int64
    dtype: object





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
      <th>user_id</th>
      <th>car_id</th>
      <th>start_time</th>
      <th>end_time</th>
      <th>num_riders</th>
      <th>region</th>
      <th>num_near_misses</th>
      <th>price</th>
      <th>rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>9</td>
      <td>spiderman</td>
      <td>2018-10-02 03:00:21</td>
      <td>2018-10-02 03:08:19</td>
      <td>3</td>
      <td>sf</td>
      <td>0</td>
      <td>3.25</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>12</td>
      <td>superman</td>
      <td>2018-10-02 03:01:30</td>
      <td>2018-10-02 03:09:16</td>
      <td>2</td>
      <td>sf</td>
      <td>0</td>
      <td>2.95</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>hulk</td>
      <td>2018-10-02 03:01:45</td>
      <td>2018-10-02 03:09:15</td>
      <td>2</td>
      <td>sf</td>
      <td>0</td>
      <td>3.22</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10</td>
      <td>spiderman</td>
      <td>2018-10-02 03:02:08</td>
      <td>2018-10-02 03:15:34</td>
      <td>3</td>
      <td>sf</td>
      <td>0</td>
      <td>2.29</td>
      <td>5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>9</td>
      <td>scarecrow</td>
      <td>2018-10-02 03:02:13</td>
      <td>2018-10-02 04:02:47</td>
      <td>4</td>
      <td>sf</td>
      <td>0</td>
      <td>2.93</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.describe()
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
      <th>user_id</th>
      <th>num_riders</th>
      <th>num_near_misses</th>
      <th>price</th>
      <th>rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1033.000000</td>
      <td>1033.000000</td>
      <td>1033.000000</td>
      <td>1033.000000</td>
      <td>1033.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>10.220716</td>
      <td>3.100678</td>
      <td>0.029042</td>
      <td>3.324985</td>
      <td>3.745402</td>
    </tr>
    <tr>
      <th>std</th>
      <td>5.906874</td>
      <td>1.445901</td>
      <td>0.251216</td>
      <td>1.224522</td>
      <td>1.063871</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>2.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>5.000000</td>
      <td>2.000000</td>
      <td>0.000000</td>
      <td>2.490000</td>
      <td>3.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>10.000000</td>
      <td>3.000000</td>
      <td>0.000000</td>
      <td>2.950000</td>
      <td>3.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>15.000000</td>
      <td>4.000000</td>
      <td>0.000000</td>
      <td>3.410000</td>
      <td>5.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>20.000000</td>
      <td>5.000000</td>
      <td>5.000000</td>
      <td>6.500000</td>
      <td>5.000000</td>
    </tr>
  </tbody>
</table>
</div>



# Feature Engineering

## Trip duration
Also fixes the "travel back in time" issue mentioned above


```python
df['duration'] = df['end_time'] - df['start_time']
df.loc[df['duration'].dt.days < 0, 'duration'] += datetime.timedelta(days=1)
df['duration_sec'] = df.duration.dt.seconds
```

## Is trip short or long
We saw the <30min vs. >1h difference above


```python
df['is_long'] = df['duration'] > datetime.timedelta(hours=1)
df.drop('duration', 1, inplace=True)
```

## Is pool
More than 1-rider


```python
df['is_pool'] = df.num_riders > 1
```

## Has at least one near miss
Just binary: did a near miss happen during this trip, to discount the ones where as much as 5 happened


```python
df['has_near_miss'] = df.num_near_misses > 0
```

## Total price
Assuming `price: the price that the rider paid to get on the ride` is per rider


```python
df['total_price'] = df.num_riders * df.price
```

## DC vs. Marvel
Some cars are Avengers (higher ratings), some are Justice League (lower ratings, and superman), and some are neither


```python
df['is_avenger'] = df.car_id.isin(['hulk', 'ironman', 'spiderman', 'venom'])
df['is_justice_league'] = df.car_id.isin(['batman', 'superman'])
```

## Normalize ratings by user's average
> Different rating scales. This problem is related to the fact that "conservative" users tend to assign items to a narrow range of rating categories whereas "liberal" users tend to assign items to a wide range of rating categories. To account for this factor, the ratings of each user are divided by the variance in his ratings.

in Jin, Rong, and Luo Si. "A study of methods for normalizing user ratings in collaborative filtering." Proceedings of the 27th annual international ACM SIGIR conference on Research and development in information retrieval. ACM, 2004.


```python
df = df.join(df.groupby('user_id').rating.agg(['mean', 'std']), 'user_id')
df['rating_normed'] = (df['rating'] - df['mean']) / df['std']
df.drop(['mean', 'std'], inplace=True, axis=1)
```

For example, the first 4 trips on file all have a rating of 5. But after weighting by the individual user's rating behavior, we can see that a rating of 5 by user #12 is much more meaningful than the same value rating by other users. That's because user #12 doesn't rate particular high on average (compared to other users) and also rates on a narrower range. In other words, user #12 might not give 5-star ratings as often as other users


```python
print(df.groupby('user_id').rating.agg(['mean', 'std']).loc[[9,12,3,10]])
df[['user_id', 'car_id', 'rating', 'rating_normed']].head(4)
```

                 mean       std
    user_id                    
    9        3.727273  1.245711
    12       3.777778  0.974420
    3        4.000000  1.036375
    10       3.763636  1.104932





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
      <th>user_id</th>
      <th>car_id</th>
      <th>rating</th>
      <th>rating_normed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>9</td>
      <td>spiderman</td>
      <td>5</td>
      <td>1.021687</td>
    </tr>
    <tr>
      <th>1</th>
      <td>12</td>
      <td>superman</td>
      <td>5</td>
      <td>1.254307</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>hulk</td>
      <td>5</td>
      <td>0.964901</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10</td>
      <td>spiderman</td>
      <td>5</td>
      <td>1.118950</td>
    </tr>
  </tbody>
</table>
</div>



## Binary ratings
Good vs Bad


```python
df['top_rating'] = df.rating >= 4
df['bad_rating'] = df.rating <= 2
```

## Night and Day
Visibility is worst at night, but traffic could be worst during the day. Note: use just `start_time` for simplicity.


```python
# helper function
between_time = df.set_index('start_time').index.indexer_between_time
```


```python
df.loc[between_time('7:06','18:50'),'is_day'] = True
df.is_day.fillna(False, inplace=True)
```

## Rush hour
More accidents? Lower rating? Oct 2nd was a Tuesday (i.e. week-day). Note: use only `start_time` for simplicity.


```python
df.loc[np.append(between_time('7:00','9:00'), between_time('16:00','18:00')), 'is_rush_hour'] = True
df.is_rush_hour.fillna(False, inplace=True)
```

## Hours


```python
df['start_time_hour'] = df.start_time.dt.hour
df['end_time_hour'] = df.end_time.dt.hour
```

## Dummy variables
We need to convert `user_id`, `car_id`, `region`, and dates/hours to binary. `user_id` is numerical, but it has no real meaning (e.g. user_id 7 > user_id 2 doesn't make sense). Same for hours (they're circular, as in the clock resets at midnight, and so they're not really numerical). `car_id` and `region` are categorical, so we need to encode them. And sklearn models don't work with timestamps. There's multiple ways to do that, the simplest one being adding an indicator variable for each category. The pandas library has a very simple method to do this, called [pd.get_dummies](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.get_dummies.html)

NOTE: this method of encode these categorical features as binary vectors is also known as [one-hot encoding](http://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html).


```python
cat_cols = ['user_id', 'car_id', 'region', 'start_time_hour', 'end_time_hour']
df = pd.concat([df, pd.get_dummies(df[cat_cols].astype(str))], axis=1)
df.drop(cat_cols + ['start_time', 'end_time'], inplace=True, axis=1)
```

# Response Variables
As a business, we're interested in maximizing user satisfaction (`rating`), revenue (`price` or total rides), and safety (minimize `num_near_misses`). Let's define those as our dependent variables

## Linear correlations
As a first pass, we can check which features correlate with our variables of interest. Not ideal for the binary variables we just created above, but it will give us a sense of direction.

## Safety
As we identified before, long rides are associated with higher number of near misses. These are also associated with bad ratings, but it's hard to know if it's causation or just correlation (i.e. is the bad rating because of the near misses or because the ride was too long? Probably the latter – see below). We've also noted how Justice League cars have a poorer safety record (mostly `superman`).


```python
corr = df.corr()['num_near_misses']
corr[corr.abs() > 0.05].sort_values(ascending=False)
```




    num_near_misses      1.000000
    has_near_miss        0.844942
    is_long              0.507523
    duration_sec         0.442549
    bad_rating           0.374386
    car_id_superman      0.257468
    is_justice_league    0.168622
    start_time_hour_9    0.133089
    end_time_hour_10     0.122517
    user_id_7            0.083013
    end_time_hour_1      0.082610
    user_id_5            0.051970
    start_time_hour_5    0.051970
    is_day               0.051914
    is_avenger          -0.087140
    top_rating          -0.114657
    rating_normed       -0.262674
    rating              -0.280487
    Name: num_near_misses, dtype: float64



Just to make it clear how strong the 0.5 correlation above with long rides really is, here we show again how **near misses only happen on long rides**


```python
df.groupby('is_long').num_near_misses.sum()
```




    is_long
    False     0
    True     30
    Name: num_near_misses, dtype: int64



Contrary to what I was expecting, near misses seem to happen slightly more often during the day then at night, but honestly the correlation is so low that it could just be random. We can look at it another way to confirm that maybe there is something there


```python
df.groupby('is_day').num_near_misses.mean()
```




    is_day
    False    0.014831
    True     0.040998
    Name: num_near_misses, dtype: float64



Rush hour doesn't seem to play a role in the near misses, though..


```python
df.groupby('is_rush_hour').num_near_misses.mean()
```




    is_rush_hour
    False    0.028302
    True     0.032432
    Name: num_near_misses, dtype: float64



Finally, note how, even though user #7 has the most number of near misses, most happened in just one trip (5). In contrast, user #17 has the most trips with at least one near miss. Regardless, I think this is just random. We would need a lot more data to conclude that the user is at fault, I believe.


```python
corr = df.corr()['has_near_miss']
corr[corr.abs() > 0.05].sort_values(ascending=False)
```




    has_near_miss        1.000000
    num_near_misses      0.844942
    is_long              0.600660
    duration_sec         0.515784
    bad_rating           0.443090
    car_id_superman      0.242998
    is_justice_league    0.160524
    end_time_hour_1      0.157864
    start_time_hour_9    0.147402
    end_time_hour_10     0.123093
    user_id_17           0.086282
    user_id_5            0.078156
    end_time_hour_8      0.075231
    start_time_hour_7    0.071132
    is_day               0.067708
    car_id_hulk         -0.058385
    is_avenger          -0.087264
    top_rating          -0.135698
    rating_normed       -0.295583
    rating              -0.319539
    Name: has_near_miss, dtype: float64



## User satisfaction 
Bad ratings come mostly from long ride times (note the associated correlation with near misses mentioned above). Avenger cars have slightly higher rating, but negligible (except `hulk`, which does seem more popular). Also slightly higher ratings for south sf compared to sf


```python
corr = df.corr()['rating']
corr[corr.abs() > 0.05].sort_values(ascending=False)
```




    rating                1.000000
    rating_normed         0.987550
    top_rating            0.900131
    car_id_hulk           0.089508
    user_id_15            0.066774
    end_time_hour_3       0.060018
    region_south sf       0.059807
    start_time_hour_20    0.059015
    price                 0.058814
    start_time_hour_21    0.057712
    user_id_3             0.056779
    is_avenger            0.055162
    end_time_hour_20      0.053339
    end_time_hour_15     -0.050766
    user_id_20           -0.052021
    end_time_hour_10     -0.056287
    region_sf            -0.059807
    is_pool              -0.060138
    is_day               -0.069758
    end_time_hour_16     -0.071218
    car_id_superman      -0.078322
    is_justice_league    -0.090285
    end_time_hour_1      -0.093004
    start_time_hour_15   -0.093602
    total_price          -0.119216
    num_riders           -0.126945
    num_near_misses      -0.280487
    has_near_miss        -0.319539
    is_long              -0.441300
    bad_rating           -0.558736
    duration_sec         -0.570826
    Name: rating, dtype: float64



Again just making it explicit how bad ratings (2 or below) only happen on long rides


```python
df.groupby('bad_rating').is_long.sum().astype(int)
```




    bad_rating
    False     0
    True     51
    Name: is_long, dtype: int64



More riders also seems to lead to lower ratings


```python
df.groupby('bad_rating').num_riders.mean()
```




    bad_rating
    False    2.972428
    True     4.444444
    Name: num_riders, dtype: float64



Particularly we see a lot of 2-star ratings for rides with 5 people


```python
df.groupby('rating').num_riders.mean()
```




    rating
    1    3.000000
    2    4.756757
    3    2.958237
    4    2.851351
    5    3.038462
    Name: num_riders, dtype: float64



That's why we recommend to abolish those. 4-rider trips are actually the highest rated!


```python
df.groupby('bad_rating').num_riders.value_counts()
```




    bad_rating  num_riders
    False       3             195
                1             193
                2             188
                4             186
                5             181
    True        5              69
                3               8
                2               6
                4               4
                1               3
    Name: num_riders, dtype: int64




```python
df.groupby('num_riders').rating.mean()
```




    num_riders
    1    3.877551
    2    3.804124
    3    3.822660
    4    3.915789
    5    3.404000
    Name: rating, dtype: float64



Finally, notice how `user_id_15` and `user_id_20` correlate with `rating` above, but that goes way when we normalize them by user (by definition)


```python
corr = df.corr()['rating_normed']
corr[corr.abs() > 0.05].sort_values(ascending=False)
```




    rating_normed         1.000000
    rating                0.987550
    top_rating            0.894602
    car_id_hulk           0.079262
    start_time_hour_20    0.066842
    region_south sf       0.062427
    start_time_hour_21    0.061604
    end_time_hour_20      0.057487
    end_time_hour_3       0.054804
    price                 0.054438
    car_id_joker          0.050860
    is_pool              -0.054014
    end_time_hour_15     -0.055230
    region_sf            -0.062427
    is_day               -0.068554
    end_time_hour_16     -0.073066
    car_id_superman      -0.074399
    is_justice_league    -0.085559
    start_time_hour_15   -0.095782
    end_time_hour_1      -0.097436
    total_price          -0.117511
    num_riders           -0.123936
    num_near_misses      -0.262674
    has_near_miss        -0.295583
    is_long              -0.430936
    bad_rating           -0.544658
    duration_sec         -0.563857
    Name: rating_normed, dtype: float64



In contrast, the region correlation (i.e. south sf having slighter ratings then sf) seems to hold. Robin also seems to drive more often in the south than sf. The opposite is true for Hulk. South sf rides are also very slightly shorter than sf, but that's probably noise (or lack of traffic)


```python
corr = df.corr()['region_south sf']
corr[corr.abs() > 0.05].sort_values(ascending=False)
```




    region_south sf       1.000000
    top_rating            0.085398
    car_id_robin          0.082018
    rating_normed         0.062427
    start_time_hour_17    0.062111
    rating                0.059807
    end_time_hour_3       0.057851
    user_id_7             0.051315
    end_time_hour_8      -0.050205
    is_long              -0.052997
    car_id_hulk          -0.062713
    duration_sec         -0.064832
    bad_rating           -0.071844
    region_sf            -1.000000
    Name: region_south sf, dtype: float64



## Price
Price doesn't correlate with much besides number of riders, more concretely, with just _is pool_ or _is private_. The `bad_rating` negative correlation is because of the low rating of 5-rider pools (which correlates with `is_pool`)


```python
corr = df.corr()['price']
corr[corr.abs() > 0.05].sort_values(ascending=False)
```




    price              1.000000
    user_id_11         0.063046
    rating             0.058814
    rating_normed      0.054438
    user_id_14         0.050352
    duration_sec      -0.056874
    car_id_superman   -0.057057
    end_time_hour_8   -0.061311
    is_long           -0.064199
    user_id_19        -0.072564
    bad_rating        -0.104411
    total_price       -0.274264
    num_riders        -0.636697
    is_pool           -0.933252
    Name: price, dtype: float64



# Feature importance
Another (perhaps better) way to measure feature importance is to actually fit a model. We try both linear and non-linear modes, but I don't think we learn anything that we didn't know already


```python
from sklearn.feature_selection import SelectKBest, chi2, f_regression
y = df.price
X = df.drop(['price', 'total_price', 'rating_normed'], 1)
model = SelectKBest(f_regression).fit(X, y)
importances = model.scores_
pd.Series(importances, X.columns).sort_values(ascending=False)[:10]
```




    is_pool            6958.711628
    num_riders          702.887995
    bad_rating           11.363395
    user_id_19            5.457527
    is_long               4.266840
    user_id_11            4.114307
    end_time_hour_8       3.890160
    rating                3.578733
    car_id_superman       3.367384
    duration_sec          3.345690
    dtype: float64




```python
y = df.num_near_misses
X = df.drop(['num_near_misses', 'has_near_miss', 'rating_normed'], 1)
model = SelectKBest(chi2).fit(X, y)
importances = model.scores_
pd.Series(importances, X.columns).sort_values(ascending=False)[:10]
```




    duration_sec         288985.277537
    is_long                 354.297792
    bad_rating              185.138407
    car_id_superman          70.002427
    end_time_hour_1          41.551036
    start_time_hour_9        40.900721
    end_time_hour_10         32.895991
    rating                   32.047377
    is_justice_league        26.388707
    start_time_hour_5        22.275832
    dtype: float64




```python
y = df.rating
X = df.drop(['rating', 'rating_normed', 'top_rating', 'bad_rating'], 1)
model = SelectKBest(chi2).fit(X, y)
importances = model.scores_
pd.Series(importances, X.columns).sort_values(ascending=False)[:10]
```




    duration_sec         597003.227075
    num_near_misses        1326.684966
    has_near_miss           665.380868
    is_long                 608.379173
    total_price             161.340387
    num_riders               71.748309
    car_id_superman          57.821870
    end_time_hour_1          37.260980
    region_south sf          20.086362
    is_justice_league        19.763780
    dtype: float64




```python
from sklearn.ensemble import ExtraTreesClassifier

y = df.rating
X = df.drop(['rating', 'rating_normed', 'top_rating', 'bad_rating'], 1)
clf = ExtraTreesClassifier(n_estimators=50)
clf = clf.fit(X, y)
importances = clf.feature_importances_  
pd.Series(importances, X.columns).sort_values(ascending=False)[:10]
```




    duration_sec        0.145298
    total_price         0.063506
    price               0.058140
    num_riders          0.039479
    is_long             0.031658
    is_avenger          0.018490
    is_day              0.015206
    car_id_spiderman    0.013156
    car_id_hulk         0.012699
    car_id_ironman      0.012618
    dtype: float64




```python
y = df.has_near_miss
X = df.drop(['num_near_misses', 'has_near_miss', 'rating_normed'], 1)
clf = ExtraTreesClassifier(n_estimators=50)
clf = clf.fit(X, y)
importances = clf.feature_importances_  
pd.Series(importances, X.columns).sort_values(ascending=False)[:10]
```




    is_long              0.177898
    rating               0.160377
    duration_sec         0.102351
    num_riders           0.065779
    car_id_superman      0.064416
    bad_rating           0.063713
    total_price          0.050736
    is_justice_league    0.025641
    start_time_hour_9    0.022976
    user_id_1            0.015607
    dtype: float64


