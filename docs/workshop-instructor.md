# Analyzing election data with Polars and Python

## INTRO ****\*\*\*\*****

```zsh
cd ~/Desktop/hands_on_classes/20260305-thursday-analyzing-election-data-with-polars-and-python/
```

```zsh
source .venv/bin/activate
```

```zsh
jupyter lab
```

- how many people used notebooks before?
- running cells
- creating cells

```python
import polars as pl
```

- look at 2024 constituency data
- load it

```python
cs = pl.read_csv("datasets/2024_constituencies.csv")
```

```zsh
cs
```

- polars shows the types under the column headers

## SELECT ****\*\*\*\*****

```python
mps = cs.select("constituency_name", "mp_firstname", "mp_surname")
```

- Polars **never changes the dataframe in place**. It always returns the transformed data as a new object.

## DROP ****\*\*\*\*****

```python
cs.drop("constituency_name")
```

## FILTER ****\*\*\*\*****

- basic filtering (x=y)

```python
wales = cs.filter(pl.col("region_name") == "Wales")
```

- other operations:
  - `==` - Equal to
  - `<` - Less than
  - `>` - Greater than
  - `<=` - Less than or equal
  - `>=` - Greater than or equal to
  - `!=` - Not equal to

- if you wanted to find all safe seats do:

```python
safe_seats = cs.filter(pl.col("majority") > 10000)
```

- if we wanted to filter for constituencies in certain countries (uk includes four countries)

```python
great_britain = [
    "England",
    "Wales",
    "Scotland"
]

cs = cs.filter(pl.col("country_name").is_in(great_britain))
```

- negate it

```python
northern_ireland = cs.filter(~ pl.col("country_name").is_in(great_britain))
```

- can do more advanced filtering if you need to say things like "if the value includes some string"

## SORT ****\*\*\*\*****

```python
cs.sort("majority")
```

```python
cs.sort("majority", descending=True)
```

```python
cs.sort(["region_name", "constituency_name"], descending=[False, True])
```

## CHAINING ****\*\*\*\*****

- we can chain all of these methods. its accumulative
- So if we wanted to find a list of safe seats won by the Labour party, sorted from the most safe to the least safe we could

```python
safe_lab_seats = (
    cs
        .select(
            "constituency_name",
            "winning_party",
            "majority")
        .filter(pl.col("winning_party") == "Lab")
        .filter(pl.col("majority") > 10000)
        .sort("majority", descending=True)
)
```

## CREATE NEW COLS ****\*\*\*\*****

- so far we have been inspecting the data as it is. now we are going to look at how to create new columns

For example, instead of filtering for safe seats for each party, we could create a new column that shows whether a seat is "safe" for the winning party in each constituency.

```python
cs.with_columns(
    (pl.col("majority") > 10000).alias("is_safe")
)
```

- There are times when you do want to edit in place, for example our declaration_time is currently a string

```python
cs.with_columns(
    pl.col("declaration_time").str.to_datetime(time_zone="Europe/London")
)
```

You can add multiple columns at once by passing multiple expressions.

```python
cs.with_columns(
    (pl.col("majority") > 10000).alias("safe_seat"),
    pl.col("declaration_time").str.to_datetime(time_zone="Europe/London")
)
```

- These (above) will put the output of the expression as the new value. Pl.lit gives more control over the value

```python
cs.with_columns(
    pl.lit(2024).alias("election_year")
)
```

```python
cs = cs.with_columns(
    pl.when(pl.col("majority") > 10000)
      .then(pl.lit("Safe"))
      .otherwise(pl.lit("Marginal"))
      .alias("seat_type")
)
```

## GROUP BY ****\*\*\*\*****

- aggregating data with group_by

- if we wanted to know how many seats each party won we would group by "winning party" and we want to count the rows in each group

```python
seats_by_party = (
    cs
        .group_by("winning_party")
        .agg(pl.len().alias("seats_won"))
        .sort("seats_won", descending=True)
)
```

- If we wanted to do something more advanced, e.g. if we wanted to group by party but find the average majority each party got we can:

```python
av_maj_by_party = (
    cs
        .group_by("winning_party")
        .agg(pl.col("majority").mean().alias("av_maj"))
        .sort("av_maj", descending=True)
)
```

```python
summary = (
    cs
        .group_by("winning_party")
        .agg(
            pl.len().alias("seats_won"),
            pl.col("majority").mean().alias("av_maj"))
        .sort("av_maj", descending=True)
)
```

- Grouping will only return the columns that you are grouping by and the columns you have told it how to handle.

## JOINS ****\*\*\*\*****

- load other dataset, inspect it (same constituency id)

```python
winners_2019 = pl.read_csv("datasets/2019_constituency_winners.csv")
```

```python
election_comparison = cs.join(
    winners_2019,
    on="constituency_id",
    how="left"
)
```

The `how` argument specifies the type of join:

- `inner` — keep only matching rows from both dataframes
- `left` — keep all rows from the left dataframe and matching rows from the right
- `right` — keep all rows from the right dataframe and matching rows from the left
- `outer` — keep all rows from both dataframes

A left join is often safest when you want to preserve your main dataset.

- Now that we have joined the datasets we can work out how many seats changed hands from one party to another

```python
seat_changes = (
    election_comparison
        .filter(pl.col("winning_party") != pl.col("winning_party_2019"))
        .group_by("winning_party_2019", "winning_party")
        .agg(pl.len().alias("count"))
        .sort("count", "winning_party_2019", descending=True)
)
```

- Grouping by multiple - there will be a different group for each unique combination of these values.

## SAVING ****\*\*\*\*****

```python
election_comparison.write_csv("datasets/election_comparison.csv")
```

- parquet

```python
election_comparison.write_parquet("datasets/election_comparison.parquet")
```

- parquet saves datatypes and much smaller than csv
