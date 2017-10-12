# Sampling and Partition

- [Sampling and Partition](#sampling-and-partition)
    - [Sampling](#sampling)
    - [Partitions](#partitions)


## Sampling

当数据量比较大，而且没有必要进行全量计算的时候，抽取少量的数据即可进行玩耍。

**`compute`**

```r
microbenchmark::microbenchmark(
    a = track_metadata_tbl %>%
      sdf_sample(fraction = 0.01, replacement = FALSE),
    
    b = track_metadata_tbl %>%
      sdf_sample(fraction = 0.01, replacement = FALSE) %>%
      compute("test1"),
    times = 5L
  )
src_tbls(sc)
```

**`compute`耗时2倍，将结果缓存为spark DataFrame test1**

**`sdf_sample` VS `sample_frac`**

```R
microbenchmark::microbenchmark(
  a = track_metadata_tbl %>%
    sdf_sample(fraction = 0.01, replacement = FALSE, seed = 20000229),

  b = track_metadata_tbl %>%
    sample_frac(size = 0.01, replace = FALSE),
  times = 5L
)
```

**dplyr的`sample_frac`要比sdf_sample 快很多**

## Partitions

用于划分训练集和测试集

```r
partitioned <- track_metadata_tbl %>%
  # Partition into training and testing sets
  sdf_partition(training = 0.7, testing = 0.3)

class(partitioned)

#> list

# Get the dimensions of the training set
dim(partitioned[["training"]])

# Get the dimensions of the testing set
dim(partitioned[["testing"]])
```

