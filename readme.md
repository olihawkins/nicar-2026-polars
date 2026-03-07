# Analyzing election data with Polars and Python

*By Eade Hemingway and Oliver Hawkins*

This is the repo for the workshop [Analyzing election data with Polars and Python](https://schedules.ire.org/nicar-2026/#/session/1061), which was delivered at NICAR 2026 in Indianapolis on the 5th of March. 

## Setup

At NICAR, the lab computers were already set up with the relevant software. You will need to manage this setup yourself to run the code on your own computer.

The workshop requires a Python project folder with a virtual environment that has `ipython`, `jupyterlab` and `polars` installed. This repo uses [uv](https://docs.astral.sh/uv/) to create the virtual environment and to install the packages and expects Python 3.13 or later. If you have `uv` installed you should be able to get set up by doing the following. 

- Clone the repository and run `uv sync` inside the project directory to set up the project
- Activate the environment with `source .venv/bin/activate`
- Launch JupyterLab with `jupyter lab`

If everything has installed correctly, JupyterLab should open in your default web browser and you are ready to begin.

If you don't use `uv` or you want to figure out how to set things up from scratch, see the [Environment](docs/environment.md) documentation for more information.

## Contents

### Intro

1. [What is Polars?](#1-what-is-polars)
2. [Why use Polars?](#2-why-use-polars)

### Setting up

3. [Launching a notebook](#3-launching-a-notebook)
4. [Importing Polars](#4-importing-polars)
5. [Our datasets](#5-our-datasets)
6. [Loading data](#6-loading-data)

### Working with data

7. [Selecting columns](#7-selecting-columns)
8. [Filtering rows](#8-filtering-rows)
9. [Sorting rows](#9-sorting-rows)
10. [Chaining method calls](#10-chaining-method-calls)
11. [Adding new columns](#11-adding-new-columns)
12. [Aggregating data](#12-aggregating-data)
13. [Joins](#13-joins)
14. [Saving datasets](#14-saving-datasets)

---

## Intro

### 1. What is Polars?

[Polars](https://pola.rs/) is a dataframe library for Python.

Dataframes are a way of representing tabular data that makes the data easy to transform and analyse.

Dataframes are ideal for storing **tidy data**.

A tidy dataset is a table where:

- Each row is an observation
- Each column is variable
- Each cell is an individual value

### 2. Why use Polars?

Doesn't Python already have a dataframe library?

- The first dataframe library for Python was called Pandas
- It's still widely used but some people find it awkward to work with
- Polars is newer and has had the opportunity to learn from Pandas' mistakes

The main advantages of Polars are:

- **Speed** - It's written in Rust so it's fast and memory efficient
- **Type safe** - It has a rich explicit type system that protects against bugs
- **Easy to use** - It has a coherent, friendly and memorable API

Polars doesn't copy the syntax of R's [tidyverse](https://tidyverse.org/), but it feels similary logical and intuitive. We find it more enjoyable to use than Pandas.

---

## Setting up

### 3. Launching a notebook

We are going to work with Polars in a Jupyter Notebook.

If you're using a NICAR Mac laptop, open the terminal app in the dock and paste the following command.

```zsh
cd ~/Desktop/hands_on_classes/20260305-thursday-analyzing-election-data-with-polars-and-python/
```

Then activate the Python environment.

```zsh
source .venv/bin/activate
```

Then start the Jupyter server.

```zsh
jupyter lab
```

This should open the JupyterLab interface in a web browser.

Have a look at [Environment](/docs/environment.md) if you want to learn how to set up a Python environment from scratch.

### 4. Importing polars

We haven't provided a pre-filled notebook because we want you to make one from scratch.

Create a new notebook by choosing `Notebook` in the launcher.

If you haven't used one before, a notebook is an interactive programming enviornment.

You can add code to each cell and then run that code.

Add the following line of code to the first cell.

```python
import polars as pl
```

There are three ways to run the cell:

- Hit the play button in the toolbar at the top of the notebook
- Use Cmd + Enter to run the cell without moving the focus
- Use Shift + Enter to run the cell and move the focus to the next cell

Running this cell will import the `polars` package under the name `pl`.

### 5. Our datasets

The data files for the workshop are included in the [datasets](/datasets) directory. These contain the results of the 2024 UK general election. The electoral system used in UK general elections is called `first-past-the-post` and it's fairly straightforward:

- Candidates compete to become the elected Member of Parliament (MP) in one of 650 constituencies
- A candidate can only stand for one party in one constituency
- Every voter casts one vote for one candidate in one constituency
- The candidate with the most votes in each constituency wins that seat

The data files we will use in this workshop are:

- `2024_constituencies.csv` - A table of the election results with one row per constituency
- `2019_constituency_winners.csv` - A table showing which party won each constituency at the previous election in 2019

During the workshop we will answer various questions about the election results by transforming and combining the data from these datasets.

### 6. Loading data

Let's start by loading the main dataset: `2024_constituencies.csv`.

Create a new cell and then add and run the following line of code:

```python
cs = pl.read_csv("datasets/2024_constituencies.csv")
```

This loads the CSV data into the `cs` variable as a dataframe.

Add another cell and just include the variable name.

```zsh
cs
```

Putting the name of a variable on the last line of a cell will print the contents of the variable.

Notice that while Polars has correctly guessed the types of most of the columns, the `declaration_time` column has been loaded as a string and not a datetime. We will fix that later.

---

## Working with data

### 7. Selecting columns

Use the `select` method to select only certain columns from the dataframe.

You can provide each column name as an argument like this.

```python
mps = cs.select("constituency_name", "mp_firstname", "mp_surname")
```

If you now print the `mps` and `cs` dataframes you will see that the `cs` dataframe hasn't changed.

Polars **never changes the dataframe in place**. It always returns the transformed data as a new object.

If you want to remove columns, you can use the `drop` function in the exact same way.

```python
no_mps = cs.drop("mp_firstname", "mp_surname", "mp_gender")
```

### 8. Filtering rows

Selecting a subset of the rows is a more complicated operation than selecting columns, because rows are filtered according to criteria that you provide.

To use the `filter` method of a dataframe, you will need to use a function called `pl.col` to refer to the columns you care about.

Let's start with a simple example.

Each of the 650 constituencies in the dataset belongs to a region. Let's find all constituencies in Wales.

```python
wales = cs.filter(pl.col("region_name") == "Wales")
```

The comparison operators you use to `filter` are the same as those used in most programming langauges:

- `==` - Equal to
- `<` - Less than
- `>` - Greater than
- `<=` - Less than or equal
- `>=` - Greater than or equal to
- `!=` - Not equal to

In our dataset their is a column called `majority`. This is the number of votes the winning candidate won over the second place candidate. The size of the majority is often taken as a measure of how safe a seat is. Let's filter for all constituencies where the majority is greater than ten thousand.

```python
safe_seats = cs.filter(pl.col("majority") > 10000)
```

#### Using `is_in`

As well as making comparisons, you can test if the value in a cell belongs to a list of values using the `is_in` method of `pl.col`. 

Let's filter our main dataset to include just the constituencies in Great Britain. This will exclude the constituencies in Northern Ireland, which has a different set of political parties to the rest of the UK and is generally analysed separately.

```python
great_britain = [
    "England", 
    "Wales", 
    "Scotland"
]

cs = cs.filter(pl.col("country_name").is_in(great_britain))
```

#### Using `~` to negate the conditioin

Use a tilde `~` at the start of the `filter` expression to find all rows that don't meet the criteria. For example, this will find only the constituencies in Northern Ireland.


```python
northern_ireland = cs.filter(~ pl.col("country_name").is_in(great_britain))
```

### 9. Sorting rows

Use the `sort` method to sort the dataframe rows. In the simplest case you just specify the column to sort by.

```python
cs.sort("majority")
```

The sort order is ascending by default. To sort descending, set the `descending` argument to `True`.

```python
cs.sort("majority", descending=True)
```

To sort by more than one column, provide a list of column names. The `descending` argument also takes a list.

```python
cs.sort(["region_name", "constituency_name"], descending=[False, True])
```

### 10. Chaining method calls

You can combine several Polars operations by chaining method calls. For example we can:

- `select` the `constituency_name`, `winning_party` and `majority` columns
- `filter` for seats where the `winning_party` was "Lab"
- `filter` for seats where the `majority` was over ten thousand
- `sort` by descending `majority`

This will give us a list of safe seats won by the Labour party, sorted from the most safe to the least safe.

Because Python uses whitespace for formatting, the neatest way to type this is to wrap the chain in parentheses.

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

### 11. Adding new columns

So far we have selected, filtered and sorted data.

But what if we want to create a new column or modify an existing one?

Use the `with_columns` method.

Like `filter`, this works using expressions built with `pl.col`.

#### Using `with_columns`

For example, instead of filtering for safe seats for each party, we could create a new column that shows whether a seat is "safe" for the winning party in each constituency.

```python
cs.with_columns(
    (pl.col("majority") > 10000).alias("is_safe")
)
```

This creates a new column called `is_safe` containing `True` or `False` values.

The expression used in this example should be familiar, because it is the same one we used to filter the dataframe previously.

But `with_columns` accepts a much wider range of expressions than `filter`, which only accepts expressions that return `True` or `False`.

#### Understanding `alias`

The `alias` method of a Polars expression is used to provide the name of the column that should be created from the expression.

#### Transforming columns in place

You can also transform existing columns.

If you don't call `alias` at the end of your expression, the new data will replace the values of the column used in your expression (or the first column, if there is more than one column).

This means that if you want to modify a column in place, you can just leave off `alias`.

For example, if we want to convert the `declaration_time` column from a string to a datetime we can use `str.to_datetime`.

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

#### Using `pl.lit`

Sometimes you want to create a column that contains the same value in every row.

To do that, use `pl.lit`, which stands for “literal”.

For example, suppose we want to label this dataset as the 2024 election.

```python
cs.with_columns(
    pl.lit(2024).alias("election_year")
)
```

This creates a new column where every row contains the value `2024`.

#### Using `pl.when`

`pl.lit` is especially useful when building conditional columns with `pl.when`.

For example:

```python
cs = cs.with_columns(
    pl.when(pl.col("majority") > 10000)
      .then(pl.lit("Safe"))
      .otherwise(pl.lit("Marginal"))
      .alias("seat_type")
)
```

This creates a new column called `seat_type` that classifies seats as “Safe” or “Marginal”.

### 12. Aggregating data

Often we want to summarise data, rather than look at individual rows.

For example:

- How many seats did each party win?
- What was the average share of the vote for the Labour party by region?

To do this, use `group_by` and `agg` (short for aggregate).

#### Counting rows by group

Let’s count how many constituencies each party won.

```python
seats_by_party = (
    cs
        .group_by("winning_party")
        .agg(pl.len().alias("seats_won"))
        .sort("seats_won", descending=True)
)
```

`pl.len()` counts the number of rows in each group.

#### Calculating aggregate statistics

We can also calculate statistics. 

For example, let's calculate the average majority by winning party.

```python
av_maj_by_party = (
    cs
        .group_by("winning_party")
        .agg(pl.col("majority").mean().alias("av_maj"))
        .sort("av_maj", descending=True)
)
```

You can calculate multiple summary statistics at once.

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

### 13. Joins

Often the data we need is spread across multiple datasets. Let's join these two:

- `2024_constituencies.csv`
- `2019_constituency_winners.csv`

To combine datasets, use a `join`.

First, load the dataset that summarises the constituency winners in 2019.

```python
winners_2019 = pl.read_csv("datasets/2019_constituency_winners.csv")
```

Both datasets contain a column called `constituency_id`.

We can join them like this:

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

After joining the dataframes, we can work out how many seats changed hands from one party to another.

```python
seat_changes = (
    election_comparison
        .filter(pl.col("winning_party") != pl.col("winning_party_2019"))
        .group_by("winning_party_2019", "winning_party")
        .agg(pl.len().alias("count"))
        .sort("count", "winning_party_2019", descending=True)
)
```

### 14. Saving datasets

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
