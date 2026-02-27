# Analyzing election data with Polars and Python

## Contents

1. [What is Polars?](#1-what-is-polars)
2. [Why use Polars?](#2-why-use-polars)
3. [Setting up](#3-setting-up)
4. [Importing Polars](#4-importing-polars)
5. [Our datasets](#5-our-datasets)
6. [Loading data](#6-loading-data)
7. [Changing the display configuration](#7-changing-the-display-configuration)
8. [Selecting columns](#8-selecting-columns)
9. [Filtering rows](#9-filtering-rows)
10. [Sorting rows](#10-sorting-rows)
11. [Chaining method calls](#11-chaining-method-calls)

## 1. What is Polars?

[Polars](https://pola.rs/) is a dataframe library for Python.

Dataframes are a way of representing tabular data that makes the data easy to transform and analyse. 

Dataframes are ideal for storing [tidy data](https://aeturrell.github.io/python4DS/data-tidy.html).

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
source .venv/bin/acivate
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

Running this cell will import the `polars` package under the name `pl`. 

There are three ways to run a cell:

- Hit the play button at the top
- Use Cmd + Enter to run the cell without moving the focus
- Use Shift + Enter to run the cell and move the focus to the next cell

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

Create a new cell and then add and run the following lines of code:

```python
constituencies = pl.read_csv("datasets/2024_constituencies.csv")
```
This loads the csv into the `constituencies` variable.

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
pl.Config.set_tbl_rows(8)
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

Or you can provide them as list.

```python
mp_columns = ["constituency_name", "mp_firstname", "mp_surname"]
mps = constituencies.select(mp_columns)
```

If you now print the `mps` and `constituencies` dataframe you will see that `constituencies` hasn't changed.

Polars **never changes the dataframe in place**. It always returns the transformed data as a new object.

## 9. Filtering rows

Selecting a subset of the rows is a more complicated operation than selecting columns, because rows are filtered according to criteria that you provide.

To use the `filter` method of a dataframe, you describe the criteria by using `pl.col` to refer to the columns you care about.

Let's start with a simple example.

Each of the 650 constituencies in the dataset belongs to a region. Let's find all constituencies in Wales.

```python
wales = constituencies.filter(pl.col("region_name") == "Wales")
```

The comparison operators are the same as those used in most programming langauges:

- `==` - Equal to
- `<`  - Less than
- `>`  - Greater than
- `<=` - Less than or equal
- `>=` - Greater than or equal to
- `!=` - Not equal to

In our dataset their is a column called `majority`. This is the number of votes the winning candidate won over the second place candidate. The size of the majority is often taken as a measure of how safe the seat is. See if you can filter for all constituencies where the majority is greater than ten thousand.

As well as making comparisons, you can test if the value of a particular cell is one of several values usint the `is_in` method of `pl.col`, which takes a list. For example, to find all the constituencies in the southern regions of the UK you could do this.

```python
southern_regions = ["South East", "South West"]
southern_constituencies = constituenties.filter(pl.col(region_name).is_in(southern_regions))
```

To logically negate the condition in a filter use the tilde sign `~` at the start of the expression.

```python
non_southern_regsions = constituencies.filter(~ pl.col("region_name").is_in(southern_regions))
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
