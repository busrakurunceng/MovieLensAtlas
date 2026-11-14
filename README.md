# MovieLensAtlas

MovieLensAtlas uses MovieLens 1M to explore how ratings vary by genre, whether the most-rated films also have the highest average ratings, how much viewers disagree, and when users rate films. PySpark reads the raw files, checks their quality, and groups and joins records to build the tables used in the analysis.

[Open the analysis notebook](notebooks/MovieLensAtlas.ipynb) · [Download MovieLens 1M from GroupLens](https://grouplens.org/datasets/movielens/1m/)

## Analysis Questions

The planned analysis starts with the overall distribution of 1–5 star ratings, then asks four questions:

1. **Genres:** How do rating counts, average ratings, and the share of 4–5 star ratings vary by genre? A rating contributes to each genre assigned to its film.
2. **Rankings and concentration:** How many films appear in both top-10 lists, one ranked by rating count and the other by average rating? What share of all ratings goes to the 50 most-rated films?
3. **Rating disagreement:** How different can the rating distributions be for films with similar averages? Standard deviation helps identify films to compare.
4. **Monthly activity:** How do the number of ratings and the number of distinct users who rated a film change month by month?

## Dataset

The project uses the official [MovieLens 1M dataset](https://grouplens.org/datasets/movielens/1m/) from GroupLens. The raw files use `::` as the field delimiter.

| File | What one row represents | Rows read |
| --- | --- | ---: |
| `data/ratings.dat` | One rating by a user for a film, with a timestamp | 1,000,209 |
| `data/movies.dat` | One film, with its title and genre labels | 3,883 |
| `data/users.dat` | One user, with demographic fields including an age-group code | 6,040 |

`MovieID` connects ratings to films. `UserID` connects ratings to user records if the optional age-group analysis is added. The notebook reads film titles using ISO-8859-1 (Latin-1) to preserve accented characters. See the [GroupLens README](https://files.grouplens.org/datasets/movielens/ml-1m-README.txt) for field definitions and data usage terms.

## Key Findings

The notebook has verified these data checks:

- All **1,000,209** rating records parsed successfully. Every rating was between **1 and 5**.
- Every rating had a positive timestamp. The earliest and latest recorded ratings were **April 25, 2000 at 23:05:32 UTC** and **February 28, 2003 at 17:49:50 UTC**.

- **Four stars** was the most common rating, with 348,971 ratings (34.89% of all ratings). Together, four- and five-star ratings accounted for 57.52%.

## How to Run

The notebook was tested on **Windows** with **Python 3.11**, **JDK 17**, and **PySpark 4.1.1** in local mode (`local[2]`). Install the Python and Jupyter extensions in VS Code and set `JAVA_HOME` to your JDK 17 installation.

1. Open the repository root in VS Code.
2. Download MovieLens 1M from GroupLens and extract it. Place `ratings.dat`, `movies.dat`, and `users.dat` directly inside the repository's `data/` directory.
3. Run these commands from the repository root in PowerShell:

   ```powershell
   py -3.11 -m venv .venv
   .\.venv\Scripts\python.exe -m pip install "pyspark==4.1.1" ipykernel  pandas matplotlib pyarrow
   ```

4. Open [`notebooks/MovieLensAtlas.ipynb`](notebooks/MovieLensAtlas.ipynb), select the Python kernel from `.venv`, and run the cells in order. The first test should print Spark version `4.1.1`, mode `local[2]`, and a filtered row count of `3`.

## CSV vs. Parquet

Spark ran the same query on both files: count ratings and distinct active
users for each month from May 2000 through January 2003. Both formats
returned identical results for all 33 months.

| Metric | CSV (uncompressed) | Parquet (Snappy) |
| --- | ---: | ---: |
| File size | 22,593,746 bytes | 5,311,430 bytes |
| Median read and query time (5 runs) | 1.221 s | 0.823 s |

In this local test, Parquet used less disk space and returned the same
results faster. The timing includes reading and running the query.
File creation was outside the timed portion.