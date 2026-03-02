# Analyzing election data with Polars and Python

## Contents

### Intro

1. [What is Polars?](#1-what-is-polars)
2. [Why use Polars?](#2-why-use-polars)

### Setting up

3. [Setting up](#3-setting-up)
4. [Importing Polars](#4-importing-polars)
5. [Our datasets](#5-our-datasets)
6. [Loading data](#6-loading-data)
7. [Changing the display configuration](#7-changing-the-display-configuration)

### Working with data

8. [Selecting columns](#8-selecting-columns)
9. [Filtering rows](#9-filtering-rows)
10. [Sorting rows](#10-sorting-rows)
11. [Chaining method calls](#11-chaining-method-calls)
12. [Adding new columns](#12-adding-new-columns)
13. [Aggregating data](#13-aggregating-data)
14. [Joins](#14-joins)
15. [Null handling](#15-null-handling)
16. [Saving datasets](#16-saving-datasets)

## 1. What is Polars?

[Polars](https://pola.rs/) is a dataframe library for Python.

Dataframes are a way of representing tabular data that makes the data easy to transform and analyse.

Dataframes are ideal for storing **tidy data**.

A tidy dataset is a table where:

- Each row is an observation
- Each column is variable
- Each cell is a single value

## 2. Why use Polars?

Doesn't Python already have a dataframe library?

- The first dataframe library for Python was called Pandas
- It's still widely used but some people find it awkward to work with
- Polars is newer and has had the opportunity to learn from Pandas' mistakes

The main advantages of Polars are:

- **Speed** - It's written in Rust so it's fast and memory efficient
- **Type safe** - It has a rich explicit type system that protects against bugs
- **Easy to use** - It has a coherent, friendly and memorable API

Polars doesn't copy the syntax of R's [tidyverse](https://tidyverse.org/), but it feels similary logical and intuitive. We find it more enjoyable to use than Pandas.

## 3. Setting up

We are going to work with Polars in a Jupyter Notebook.

First make sure your Python environment is activated.

```zsh
source .venv/bin/activate
```

Then start the Jupyter server.

```zsh
jupyter lab
```

This should open the JupyterLab interface in a web browser.

## 4. Importing polars

We haven't provided a pre-filled notebook because we want to you make one from scratch.

Create a new notebook by choosing `Notebook` in the launcher.

If you haven't used one before, a notebook is an interactive programming enviornment.

You can add code to each cell and then run that code.

Add the following line of code to the first cell.

```python
import polars as pl
```

There are three ways to run a cell:

- Hit the play button at the top
- Use Cmd + Enter to run the cell without moving the focus
- Use Shift + Enter to run the cell and move the focus to the next cell

Running this cell will import the `polars` package under the name `pl`.

## 5. Our datasets

The data files for the session are included in the `datasets` directory. These contain the results of the 2024 UK general election. The electoral system used in UK general elections is called `first-past-the-post` and it's fairly straightforward:

- Candidates compete to become the elected Member of Parliament (MP) in one of 650 constituencies
- A candidate can only stand for one party in one constituency
- Every voter casts one vote for one candidate in one constituency
- The candidate with the most votes in each constituency wins that seat
- The party that wins the most seats forms the next government (most of the time)

The data files are:

- `2024_constituencies.csv` - A table of the election results with one row per constituency
- `2024_candidates.csv` - A table of the election results with one row per candidate
- `2019_constituency_winners.csv` - A table showing which party won each constituency at the previous election in 2019

During the workshop we will answer various questions about the election results by transforming and combining the data from these datasets.

## 6. Loading data

Let's start by loading the main dataset: `2024_constituencies.csv`.

Create a new cell and then add and run the following line of code:

```python
constituencies = pl.read_csv("datasets/2024_constituencies.csv")
```

This loads the CSV data into the `constituencies` variable as a dataframe.

Add another cell and just include the variable name.

```zsh
constituencies
```

Putting the name of a variable on the last line of a cell will print the contents of the variable.

Notice that while Polars has correctly guessed the types of most of the columns, the `declaration_time` column has been loaded as a string and not a datetime. We will fix that later.

## 7. Changing the display configuration

By default Polars prints only ten rows from the dataframe: the first five and the last five.

To change this limit you can set the Polars configuration like this:

```python
pl.Config.set_tbl_rows(20)
```

In some environments (e.g. IPython) it also shows only some of the columns. You can change that setting too if you need to.

```python
pl.Config.set_tbl_cols(10)
```

## 8. Selecting columns

Use the `select` method to select only certain columns from the dataframe.

You can provide each column name as an argument like this.

```python
mps = constituencies.select("constituency_name", "mp_firstname", "mp_surname")
```

Or you can provide them as a list.

```python
mp_columns = ["constituency_name", "mp_firstname", "mp_surname"]
mps = constituencies.select(mp_columns)
```

If you now print the `mps` and `constituencies` dataframe you will see that `constituencies` hasn't changed.

Polars **never changes the dataframe in place**. It always returns the transformed data as a new object.

## 9. Filtering rows

Selecting a subset of the rows is a more complicated operation than selecting columns, because rows are filtered according to criteria that you provide.

To use the `filter` method of a dataframe, you will need to use `pl.col` to refer to the columns you care about.

Let's start with a simple example.

Each of the 650 constituencies in the dataset belongs to a region. Let's find all constituencies in Wales.

```python
wales = constituencies.filter(pl.col("region_name") == "Wales")
```

The comparison operators are the same as those used in most programming langauges:

- `==` - Equal to
- `<` - Less than
- `>` - Greater than
- `<=` - Less than or equal
- `>=` - Greater than or equal to
- `!=` - Not equal to

In our dataset their is a column called `majority`. This is the number of votes the winning candidate won over the second place candidate. The size of the majority is often taken as a measure of how safe the seat is. 

### Try it yourself

See if you can filter for all constituencies where the majority is greater than ten thousand.

### Using `is_in`

As well as making comparisons, you can test if the value of a particular cell is one of several values usint the `is_in` method of `pl.col`, which takes a list. For example, to find all the constituencies in the southern regions of the UK you could do this.

```python
northern_regions = [
    "North East", 
    "North West", 
    "Yorkshire and The Humber"
]

northern_constituencies = constituencies.filter(pl.col("region_name").is_in(northern_regions))
northern_constituencies
```

### Using `~` to negate the conditioin

Use a tilde `~` at the start of the `filter` expression to find all rows that don't meet the criteria.

You can read is as "not *{the filter criteria}*".

```python
non_southern_regsions = constituencies.filter(~ pl.col("region_name").is_in(northern_regions))
```

## 10. Sorting rows

Use the `sort` method so sort the rows. In the simplest case you just specify the column to sort by.

```python
constituencies.sort("majority")
```

The sort order is ascending by default. To sort descending, set the `descending` argument to `True`.

```python
constituencies.sort("majority", descending=True)
```

To sort by more than one column, provide a list of column names. The `descending` argument also takes a list.

```python
constituencies.sort(["region_name", "constituency_name"], descending=[True, False])
```

## 11. Chaining method calls

You can combine several Polars operations by chaining method calls. For example we can:

- `select` the `constituency_name`, `winning_party` and `majority` columns
- `filter` for seats where the `winning_party` was "Lab"
- `filter` for seats where the `majority` was over ten thousand
- `sort` by descending `majority`

This will give us a list of safe Labour seats, sorted from the most safe to the least safe.

Because Python uses whitespace for formatting, the neatest way to type this is to wrap the chain in parentheses.

```python
safe_lab_seats = (
    constituencies
        .select(
            "constituency_name",
            "winning_party",
            "majority")
        .filter(pl.col("winning_party") == "Lab")
        .filter(pl.col("majority") > 10000)
        .sort("majority", descending=True)
)
```

One other helpful method is `with_row_index`. This adds a column to the start of your dataframe indexing the rows. You can use the `offset` argument to specify the start index.

```python
safe_lab_seats = (
    constituencies
        .select(
            "constituency_name",
            "winning_party",
            "majority")
        .filter(pl.col("winning_party") == "Lab")
        .filter(pl.col("majority") > 10000)
        .sort("majority", descending=True)
        .with_row_index(offset=1)
)
```

## 12. Adding new columns

So far we have selected, filtered and sorted data.

But what if we want to create a new column or modify an existing one?

Use the `with_columns` method.

Like `filter`, this works using Polars expressions and `pl.col`.

### Using expressions in `with_columns`

For example, instead of filtering for safe seats for each party, we could create a new column that shows whether a seat is "safe" for the winning party in each constituency.

```python
constituencies.with_columns(
    (pl.col("majority") > 10000).alias("is_safe")
)
```

This creates a new column called `is_safe` containing `True` or `False` values.

The expression used in this example should be familiar, because it is the same one we used to filter the dataframe.

But `with_columns` accepts a much wider range of expressions than `filter`, which only accepts expressions that return True or False.

### Understanding `alias`

The `alias` method of a Polars expression is used to provide the name of the column that should be created from the expression.

### Transforming columns in place

If you don't call `alias` at the end of your expression, the new data will replace the values of the first column in the expression.

This means that if you want to modify a column in place, you can just leave off `alias`.

You can also transform existing columns. For example, suppose we want to convert the `declaration_time` column from a string to a datetime:

```python
constituencies.with_columns(
    pl.col("declaration_time").str.to_datetime(time_zone="Europe/London")
)
```

You can add multiple columns at once by passing multiple expressions.

```python
constituencies.with_columns(
    pl.col("declaration_time").str.to_datetime(time_zone="Europe/London"),
    (pl.col("valid_votes") / pl.col("electorate") * 100).alias("turnout")
)
```

### Using `pl.lit`

Sometimes you want to create a column that contains the same value in every row.

To do that, use `pl.lit`, which stands for “literal”.

For example, suppose we want to label this dataset as the 2024 election.

```python
constituencies.with_columns(
    pl.lit(2024).alias("election_year")
)
```

This creates a new column where every row contains the value 2024.

### Using `pl.when`

`pl.lit` is especially useful when building conditional columns with `pl.when`.

For example:

```python
constituencies = constituencies.with_columns(
    pl.when(pl.col("majority") > 10000)
      .then(pl.lit("Safe"))
      .otherwise(pl.lit("Marginal"))
      .alias("seat_type")
)
```

This creates a new column called `seat_type` that classifies seats as “Safe” or “Marginal”.

## 13. Aggregating data

Often we want to summarise data, rather than look at individual rows.

For example:

- How many seats did each party win?
- What was the average majority by region?

To do this, use `group_by` and `agg` (short for aggregate).

Let’s count how many constituencies each party won.

```python
seats_by_party = (
    constituencies
        .group_by("winning_party")
        .agg(pl.len().alias("seats_won"))
        .sort("seats_won", descending=True)
)
```

`pl.len()` counts the number of rows in each group.

We can also calculate statistics. 

For example, let's first add a column to our dataset that calculates the Labour party's share of the vote in each constituency.

```python
constituencies = constituencies.with_columns(
    (pl.col("votes_lab") / pl.col("valid_votes") * 100).alias("share_lab"),
)
```

And then let's calculate Labour's average share of the vote in each region.

```python
average_share_lab = (
    constituencies
        .group_by("region_name")
        .agg(pl.col("share_lab").mean().alias("avg_share_lab"))
        .sort("avg_share_lab", descending=True)
)
```

You can calculate multiple summary statistics at once.

```python
summary_lab = (
    constituencies
        .group_by("region_name")
        .agg(
            pl.len().alias("seats"),
            (pl.col("winning_party") == "Lab").sum().alias("seats_lab"),
            pl.col("share_lab").mean().alias("avg_share_lab"),
            pl.col("share_lab").min().alias("min_share_lab"),
            pl.col("share_lab").max().alias("max_share_lab")
        )
        .sort("avg_share_lab", descending=True)
)
```

## 14. Joins

Often the data we need is spread across multiple datasets.

We have:

- `2024_constituencies.csv`
- `2024_candidates.csv`
- `2019_constituency_winners.csv`

To combine datasets, use a join.

First, load the dataset that summarises the constituency winners in 2019.

```python
winners_2019 = pl.read_csv("datasets/2019_constituency_winners.csv")
```

Both datasets contain a column called `constituency_id`.

We can join them like this:

```python
election_comparison = constituencies.join(
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

### Try it yourself

After joining the dataframes, you can analyse changes between 2019 and 2024.

Try comparing the 2024 `winning_party` with the `winning_party_2019` to find all seats that changed hands.

## 15. Null handling

Real-world datasets often contain missing values. Polars represents these missing data values as `null`.

Our datasets do not contain any `null` values, but let's make a dataframe that contains some nulls by doing the same join as before, but with a random subset of rows from the second dataframe. 

By sampling half of the 650 rows, half the rows in our resulting dataframe will contain `null` values for the columns from `winners_2019`.

```python
has_nulls = constituencies.join(
    winners_2019.sample(645),
    on="constituency_id",
    how="left"
)
```

You can check for null values using `is_null`.

```python
has_nulls.filter(pl.col("winning_party_2019").is_null())
```

You can remove rows containing nulls using `drop_nulls`.

```python
has_nulls.drop_nulls()
```

Or fill null values with something ellse using `fill_nulls`.

```python
has_nulls.with_columns(
    pl.col("winning_party_2019").fill_null("Unknown")
)
```

When using `group_by`, `null` values are ignored in most aggregation functions like `mean`.

Being explicit about how you handle nulls helps prevent mistakes in analysis.

## 16. Saving datasets

After transforming your data, you may want to save the result.

To save a dataframe as a CSV file:

```python
election_comparison.write_csv("datasets/election_comparison.csv")
```

You can also save to other formats such as Parquet, which is often faster and more efficient:

```python
election_comparison.write_parquet("datasets/election_comparison.parquet")
```

Parquet files are **great**! They preserve the datatypes of the columns, so you don't get errors from your software having to guess the datatype by looking at the data. They are also typically much smaller than CSVs because they use efficient compression.
