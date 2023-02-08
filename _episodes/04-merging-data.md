---
title: Combining DataFrames with pandas
teaching: 20
exercises: 25
questions:
- " Can I work with data from multiple sources? "
- " How can I combine data from different data sets? "
objectives:
    - Combine data from multiple files into a single DataFrame using merge and concat.
    - Combine two DataFrames using a unique ID found in both DataFrames.
    - Employ `to_csv` to export a DataFrame in CSV format.
    - Join DataFrames using common fields (join keys).
---

In many "real world" situations, the data that we want to use come in multiple
files. We often need to combine these files into a single DataFrame to analyze
the data. The pandas package provides [various methods for combining
DataFrames](http://pandas.pydata.org/pandas-docs/stable/merging.html) including
`merge` and `concat`.

To work through the examples below, we first need to load the species and
surveys files into pandas DataFrames. The authors.csv and places.csv data can be found in the data folder. 

```python
import pandas as pd

author_link = "https://raw.githubusercontent.com/carpentries-incubator/python-humanities-lesson/gh-pages/data/authors.csv"
places_link = "https://raw.githubusercontent.com/carpentries-incubator/python-humanities-lesson/gh-pages/data/places.csv"

authors_df = pd.read_csv(author_link,
                         keep_default_na=False, na_values=[""])
authors_df

        TCP                                             Author
0    A00002                         Aylett, Robert, 1583-1655?
1    A00005  Higden, Ranulf, d. 1364. Polycronicon. English...
2    A00007             Higden, Ranulf, d. 1364. Polycronicon.
3    A00008          Wood, William, fl. 1623, attributed name.
4    A00011

places_df = pd.read_csv(places_link,
                         keep_default_na=False, na_values=[""])
places_df
    A00002                         London
0   A00005                         London
1   A00007                         London
2   A00008               The Netherlands?
3   A00011                      Amsterdam
4   A00012                         London
5   A00014                         London

```
[More about all of the read_csv options here.](http://pandas.pydata.org/pandas-docs/dev/generated/pandas.io.parsers.read_csv.html)

# Concatenating DataFrames

We can use the `concat` function in Pandas to append either columns or rows from
one DataFrame to another.  Let's grab two subsets of our data to see how this
works.

```python
# read in first 10 lines of the places table
place_sub = places_df.head(10)
# grab the last 20 rows 
place_sub_last10 = places_df.tail(20)
#reset the index values to the second dataframe appends properly
place_sub_last10 = place_sub_last10.reset_index(drop=True)
# drop=True option avoids adding new index column with old index values
```

```python
# stack the DataFrames on top of each other
vertical_stack = pd.concat([place_sub, place_sub_last10], axis=0)

# place the DataFrames side by side
horizontal_stack = pd.concat([place_sub, place_sub_last10], axis=1)
```

### Row Index Values and Concat
Have a look at the `vertical_stack` dataframe? Notice anything unusual?
The row indexes for the two data frames `place_sub` and `place_sub_last10`
have been repeated. We can reindex the new dataframe using the `reset_index()` method.

## Writing Out Data to CSV

We can use the `to_csv` command to do export a DataFrame in CSV format. Note that the code
below will by default save the data into the current working directory. We can
save it to a different folder by adding the foldername and a slash to the file
`vertical_stack.to_csv('foldername/out.csv')`. We use the 'index=False' so that
pandas doesn't include the index number for each line.

```python
# Write DataFrame to CSV
vertical_stack.to_csv('out.csv', index=False)
```

Check out your working directory to make sure the CSV wrote out properly, and
that you can open it! If you want, try to bring it back into python to make sure
it imports properly.

```python
# for kicks read our output back into python and make sure all looks good
new_output = pd.read_csv('out.csv', keep_default_na=False, na_values=[""])
```

> ## Challenge - Combine Data
>
> In the data folder, there are two catalogue data files: `1635.csv` and
> `1640.csv`. Read the data into python and combine the files to make one
> new data frame.
{: .challenge}

# Joining DataFrames

When we concatenated our DataFrames we simply added them to each other -
stacking them either vertically or side by side. Another way to combine
DataFrames is to use columns in each dataset that contain common values (a
common unique id). Combining DataFrames using a common field is called
"joining". The columns containing the common values are called "join key(s)".
Joining DataFrames in this way is often useful when one DataFrame is a "lookup
table" containing additional data that we want to include in the other.

NOTE: This process of joining tables is similar to what we do with tables in an
SQL database.

The `places.csv` file is table that contains the place and EEBO id for some titles. 
When we want to access that information, we can create a query that joins the additional 
columns of information to the author data.


## Identifying join keys

To identify appropriate join keys we first need to know which field(s) are
shared between the files (DataFrames). We might inspect both DataFrames to
identify these columns. If we are lucky, both DataFrames will have columns with
the same name that also contain the same data. If we are less lucky, we need to
identify a (differently-named) column in each DataFrame that contains the same
information.

```python
>>> authors_df.columns

Index(['TCP', 'Author'], dtype='object')

>>> places_df.columns

Index(['TCP', 'Place'], dtype='object')

```

In our example, the join key is the column containing the identifier, which is called `TCP`.

Now that we know the fields with the common TCP ID attributes in each
DataFrame, we are almost ready to join our data. However, since there are
[different types of joins](http://blog.codinghorror.com/a-visual-explanation-of-sql-joins/), we
also need to decide which type of join makes sense for our analysis.

## Inner joins

The most common type of join is called an _inner join_. An inner join combines
two DataFrames based on a join key and returns a new DataFrame that contains
**only** those rows that have matching values in *both* of the original
DataFrames.

Inner joins yield a DataFrame that contains only rows where the value being
joins exists in BOTH tables. An example of an inner join, adapted from [this
page](http://blog.codinghorror.com/a-visual-explanation-of-sql-joins/) is below:

![Inner join -- courtesy of codinghorror.com](http://blog.codinghorror.com/content/images/uploads/2007/10/6a0120a85dcdae970b012877702708970c-pi.png)

The pandas function for performing joins is called `merge` and an Inner join is
the default option:  

```python
merged_inner = pd.merge(authors_df, places_df, on='TCP')

# what's the size of the output data?
merged_inner.shape
merged_inner
```

**OUTPUT:**

```
      TCP                                             Author             Place
0  A00002                         Aylett, Robert, 1583-1655?            London
1  A00005  Higden, Ranulf, d. 1364. Polycronicon. English...            London
2  A00007             Higden, Ranulf, d. 1364. Polycronicon.            London
3  A00008          Wood, William, fl. 1623, attributed name.  The Netherlands?
4  A00011                                                NaN         Amsterdam
```

The result of an inner join of `authors_df` and `places_df` is a new DataFrame
that contains the combined set of columns from those tables. It
*only* contains rows that are the same in both the `authors_df` and `place_df` DataFrames. So, if a row in
`authors_df` has a value of `TCP` that does *not* appear in `places_df`, it will not be included in the DataFrame returned by an
inner join.  Similarly, if a row in `places_df` has a value of `TCP`
that does *not* appear in the `TCP` column of `authors_df`, that row will not
be included in the DataFrame returned by an inner join.

The result `merged_inner` DataFrame contains all of the columns from `authors`
(TCP, Person) as well as all the columns from `places_df`
(TCP, Place).

## Other join types

The pandas `merge` function supports two other join types:

* Left join: Invoked by passing `how='left'` as an argument. All* rows from the `left` DataFrame are kept, while
  rows from the `right` DataFrame without matching join key(s) values are
  discarded.
* Right (outer) join: Invoked by passing `how='right'` as an argument. Similar
  to a left join, except *all* rows from the `right` DataFrame are kept, while
  rows from the `left` DataFrame without matching join key(s) values are
  discarded.
* Full (outer) join: Invoked by passing `how='outer'` as an argument. This join
  type returns the all pairwise combinations of rows from both DataFrames; i.e.,
  the result DataFrame will `NaN` where data is missing in one of the dataframes.
  This join type is very rarely used.

# Final Challenges

> ## Challenge - Distributions
> Create a new DataFrame by joining the contents of the `authors.csv` and
> `places.csv` tables. Calculate the:
>
> 1. Number of unique places
> 2. Number of books that do not have a known place
> 3. Number of books that do not have either a known place or author
{: .challenge}
