# Machine Learning Functions

所有spark的机器学习算法均以`ml_*`开头，`ls("package:sparklyr", pattern = "^ml")`

- [Machine Learning Functions](#machine-learning-functions)
    - [Read Raw Data](#read-raw-data)
    - [Prepare Data](#prepare-data)
    - [Modeling](#modeling)
    - [Prediction](#prediction)
    - [Visualization](#visualization)
    - [Modeling2 - Random Forest](#modeling2---random-forest)
    - [Comparing Model Performance](#comparing-model-performance)

## Read Raw Data

**csv VS. parquet(/par'kei/)**

> csv files are really slow to read and write, making them unusable for large datasets. Parquet files provide a higher performance alternative. As well as being used for Spark data, parquet files can be used with other tools in the Hadoop ecosystem, like Shark, Impala, Hive, and Pig.

`spark_read_parquet(sc, "df_name", "path/to/parquet/dir")`

> this function will import the data directly into Spark, which is typically faster than importing the data into R, then using copy_to() to copy the data from R to Spark.

```r
parquet_dir

# List the files in the parquet dir
filenames <- list.files(parquet_dir, full.name = TRUE)

# Show the filenames and their sizes
tibble(
  filename = basename(filenames),
  size_bytes = file.size(filenames)
)

# Import the data into Spark
timbre_tbl <- spark_read_parquet(spark_conn,"timbre",parquet_dir)
```

> There is one more data cleaning task you need to do. The `year` column contains integers, but Spark modeling functions require real numbers. You need to convert the `year` column to numeric.

```r
track_data_tbl <- track_metadata_tbl %>%
  # Inner join to timbre_tbl
  inner_join(timbre_tbl, by = "track_id") %>%
  # Convert year to numeric
  mutate(year = as.numeric(year))
```

## Prepare Data

use user_id to partition

```r
# track_data_tbl has been pre-defined
track_data_tbl

training_testing_artist_ids <- track_data_tbl %>%
  # Select the artist ID
  select(artist_id) %>%
  # Get distinct rows
  distinct(artist_id) %>%
  # Partition into training/testing sets
  sdf_partition(training = 0.7, testing= 0.3)

track_data_to_model_tbl <- track_data_tbl %>%
  # Inner join to training partition
  inner_join(training_testing_artist_ids$training,by = "artist_id")

track_data_to_predict_tbl <- track_data_tbl %>%
  # Inner join to testing partition
  inner_join(training_testing_artist_ids$testing,by = "artist_id")
```

## Modeling

Gradient Boosted Trees:
> Gradient boosting is a technique to improve the performance of other models. The idea is that you run a weak but easy to calculate model. Then you replace the response values with the residuals from that model, and fit another model. By "adding" the original response prediction model and the new residual prediction model, you get a more accurate model. You can repeat this process over and over, running new models to predict the residuals of the previous models, and adding the results in. With each iteration, the model becomes stronger and stronger. 

> Gradient boosting with decision trees  can be used for both classification problems (where the response variable is categorical) and regression problems (where the response variable is continuous). In the regression case, as you'll be using here, the measure of how badly a point was fitted is the residual.




```r
# track_data_to_model_tbl has been pre-defined
track_data_to_model_tbl
#> "artist_id"  "year"  "timbre_means1"  "timbre_means2"  "timbre_means3"  "timbre_means4"  "timbre_means5"  "timbre_means6"  "timbre_means7"  "timbre_means8"  "timbre_means9"  "timbre_means10" "timbre_means11"  "timbre_means12"

feature_colnames <- track_data_to_model_tbl %>%
  # Get the column names
  colnames() %>%
  # Limit to the timbre columns
  str_subset(regex("^timbre"))
  # str_subset(fixed("timbre"))

gradient_boosted_trees_model <- track_data_to_model_tbl %>%
  # Run the gradient boosted trees model
  ml_gradient_boosted_trees(response = "year",
   features = feature_colnames, type = "regression")
```

## Prediction

```r
predict(a_model, testing_data)

responses <- track_data_to_predict_tbl %>%
  # Select the year column
  select(year) %>%
  # Collect the results
  collect() %>%
  # Add in the predictions
  mutate(
    predicted_year = predict(
      gradient_boosted_trees_model,
      track_data_to_predict_tbl
    )
  )
```

## Visualization

> Firstly, it's nice to draw a scatterplot of the predicted response versus the actual response, to see how they compare. Secondly, the residuals ought to be somewhere close to a normal distribution, so it's useful to draw a density plot of the residuals.

```r
# Draw a scatterplot of predicted vs. actual
ggplot(responses, aes(actual, predicted)) +
  # Add the points
  geom_point(alpha = 0.1) +
  # Add a line at actual = predicted
  geom_abline(slope = 1, intercept = 0)

residuals <- responses %>%
  # Transmute response data to residuals
  transmute(residual = predicted - actual)

# Draw a density plot of residuals
ggplot(residuals, aes(residual)) +
    # Add a density curve
    geom_density() +
    # Add a vertical line through zero
    geom_vline(xintercept = 0)
```

## Modeling2 - Random Forest

> Like gradient boosted trees, random forests are another form of _ensemble model_. That is, they use lots of simpler models (decision trees, again) and combine them to make a single better model. Rather than running the same model iteratively, random forests run lots of separate models in parallel, each on a randomly chosen subset of the data, with a randomly chosen subset of features. Then the final decision tree makes predictions by aggregating the results from the individual models.

> sparklyr's random forest function is called `ml_random_forest()`. Its usage is exactly the same as `ml_gradient_boosted_trees()`.

```r
# track_data_to_model_tbl has been pre-defined
track_data_to_model_tbl

# Get the timbre columns
feature_colnames <- track_data_to_model_tbl %>%
  colnames() %>%
  str_subset(regex("^timbre"))

# Run the random forest model
random_forest_model <- track_data_to_model_tbl %>%
  ml_random_forest(response = "year", features = feature_colnames)

# Create a response vs. actual dataset
responses <- track_data_to_predict_tbl %>%
  select(actual = year) %>%
  collect() %>% # !important
  mutate(response = predict(
    random_forest_model,
    track_data_to_predict_tbl))

# both_responses has been pre-defined
both_responses
#> A tibble: 730 x 3
#>   actual model predicted
#>    <int> <chr>     <dbl>
#> 1   1940   gbt  1951.933
#> 2   1934   gbt  1950.519

# Draw a scatterplot of predicted vs. actual
ggplot(both_responses, aes(actual, predicted, color = model)) +
  # Add a smoothed line
  geom_smooth() +
  # Add a line at actual = predicted
  geom_abline(intercept = 0, slope = 1)

# Create a tibble of residuals
residuals <- both_responses %>%
  mutate(residual = predicted - actual)

# Draw a density plot of residuals
ggplot(residuals, aes(residual, color = model)) +
    # Add a density curve
    geom_density() +
    # Add a vertical line through zero
    geom_vline(xintercept = 0)
```

## Comparing Model Performance

> Plotting gives you a nice feel for where the model performs well, and where it doesn't. Sometimes it is nice to have a statistic that gives you a score for the model. This way you can quantify how good a model is, and make comparisons across lots of models. A common statistic is the root mean square error (sometimes abbreviated to "RMSE"), which simply squares the residuals, then takes the mean, then the square root. A small RMSE score for a given dataset implies a better prediction. (By default, you can't compare between different datasets, only different models on the same dataset. Sometimes it is possible to normalize the datasets to provide a comparison between them.)

$$mse = \frac{\sum_{i =1}^{n}(\theta_i - \hat{\theta_i})^2}{n}$$

$$rmse = \sqrt{mse}$$
