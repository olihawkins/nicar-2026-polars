# nicar-2026-polars

This repo contains resources for the NICAR 2026 workshop [Analyzing election data with Polars and Python](https://schedules.ire.org/nicar-2026/#/session/1061),

## Software environment

The workshop requires a Python project folder with a virtual environment that has `ipython`, `jupyterlab` and `polars` installed. This repo uses [uv](https://docs.astral.sh/uv/) to set up the virtual environment and to install the packages, but the instructions can be adapted to use plain `pip` or `pipenv`. The Python version should be version 3.13 or later. See the [Environment](docs/environment.md) documentation for detailed notes on setting up the environment.

## Data files

The data files for the session are included in the `datasets` directory. These contain the results of the 2024 UK general election. The electoral system used in UK general elections is called "first-past-the-post" and it is fairly straightforward:

- Candidates compete to become the elected Member of Parliament (MP) in one of 650 constituencies
- A candidate can only stand for one party in one constituency
- Every voter casts one vote for one candidate in one constituency
- The candidate with the most votes in each constituency wins that seat

The party that wins the most seats forms the next government (most of the time).

The data files are:

- `2024_constituencies.csv` - A table of the election results with one row per constituency
- `2024_candidates.csv` - A table of the election results with one row per candidate
- `2019_winning_parties.csv` - A table showing which party won each constituency at the previous election in 2019

During the workshop we will answer various questions about the election results by transforming and combining the data from these datasets.