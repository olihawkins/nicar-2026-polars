# Analyzing election data with Polars and Python

## Contents

1. [What is Polars?](#1-what-is-polars)
2. [Why use Polars?](#2-why-use-polars)
3. [Setting up](#3-setting-up)
4. [Importing Polars](#4-importing-polars)
5. [Our datasets](#5-our-datasets)
6. [Loading data](#6-loading-data)
7. [Selecting columns](#7-selecting-columns)

## 1. What is Polars?

[Polars](https://pola.rs/) is a dataframe library for Python.

Dataframes are a way of representing tabular data that makes the data easy to transform and analyse. Dataframes are ideal for storing [tidy data](https://aeturrell.github.io/python4DS/data-tidy.html).

A tidy dataset is a table where:
- Each row is an observation
- Each column is variable
- Each cell is a single value

## 2. Why use Polars?

Doesn't Python already have a dataframe library?

- The first dataframe library for Python was called Pandas
- It's still widely used but some people find it awkward to work with
- Polars is newer and had the opportunity to learn from Pandas' mistakes

The main advantages of Polars are:

- **Speed** - It's written in Rust so it's fast and memory efficient
- **Types safe** - It has a rich explicit type system that protects against bugs
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

Create a new notebook by choosing `Notebook` in the launcher.

We haven't provided a prefilled notebook because we want to you make one from scratch.

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

The data files for the session are included in the `datasets` directory. These contain the results of the 2024 UK general election. The electoral system used in UK general elections is called `first-past-the-post` and it is fairly straightforward:

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

Add another cell and just include the variable name

```zsh
constituencies
```

Putting the name of a variable on the last line of a cell will print the contents of the variable.

Notice that while Polars has correctly guessed the types of most of the columns, the `declaration_time` column has been loaded as a string and not a datetime. We will fix that later.

## 7. Selecting columns





