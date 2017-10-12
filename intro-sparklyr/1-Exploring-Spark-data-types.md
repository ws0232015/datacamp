# Exploring Spark Data Types


## Some Useful Commands

- list Spark DataFrames `src_tbls(sc)`
- explort tibble structural `glimpse(a_tibble)`
- `sparklyr` function `sdf_schema(a_tibble)`

## Comparison of R and Spark Data Types

R type | Spark type
-------|-----------
logical | BooleanType
numeric | DoubleType
integer | IntegerType
character | StringType
list | ArrayType


```R
track_metadata_tbl = tbl(sc, "track_meatadata")

# schema is a list
(schema <- sdf_schema(track_metadata_tbl))

# Transform the schema
schema %>%
  lapply(function(x) do.call(tibble, x)) %>%
  bind_rows()
```

**注意： `do.call` 将list以向量的形式传进function 里面**
