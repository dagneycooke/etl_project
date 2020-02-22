# ETL Project

## Team
* Dagney Cooke
* Diana Silva
* Heain Yee

## Before you clone this repository
Add "AllMoviesDetailsCleaned.csv" from the Kaggle site for [350 000+ movies](https://www.kaggle.com/stephanerappeneau/350-000-movies-from-themoviedborg#AllMoviesDetailsCleaned.csv). 

OR You can access this file also at this [Google Drive](https://drive.google.com/drive/folders/1oxaFbyAWkC3LX9S5jFebvk79Q7vLa6ek).

## Data Sources
* [Golden Globe Awards](https://www.kaggle.com/unanimad/golden-globe-awards)
* [350 000+ movies](https://www.kaggle.com/stephanerappeneau/350-000-movies-from-themoviedborg#AllMoviesDetailsCleaned.csv)

## Project Purpose
We wanted to create a data source to enable analyses on movies and "best-picture" Golden Globe awards (and nomiations) over 20 years (1997 to 2016). We chose an end date of 2016 because our movies data contained complete data upto the year 2016.

The final data table includes the following columns:
* title (string) - title of film
* genres (string) - genre of film
* release_year (int) - year that film was released
* nom_category (string) - nomination category (NaN if film was not nominated)
* nom_year (int) - year film was nominated (NaN if film was not nominated)
* win (boolean) - whether or not film won the nomination (NaN if film was not nominated)
* popularity (float) - popularity score
* budget (float) - budget for film (NaN if information is not available)
* revenue (float) - revenue for film (NaN if information is not available)
* profit (float) - profit for film (NaN if budget or revenue information is not available)

## Extract

We found two data sources from Kaggle.  The first one was a CSV file of all Golden Globe nominees and winners from 1944 - 2020.  Our second source had multiple csv files related to 350,000 movies, and we chose to use their AllMoviesCleaned.csv file, which was pulled from themoviedb.org (it covers all movies until AUG17  - ~350k movies).  

Both of these files were read into pandas dataframes for cleaning, with the goal of being able to combine tables based on movie titles.

In both these files, we removed columns we were not interested in for.


**Golden Globes Extracted Columns**
We were interested in the following columns for the Golden Globes data: 
* year_film - year that film was released
* year_award - year film was nominated
* category - nomination category
* nominee - nominee
* film - title of film for the nominee
* win - whether or not film won the nomination

We dropped the following column:
* ceremony - Xth ceremony of nomination (int ranging 1 (first ceremony in 1944) to 77 (last ceremony in 2019))


**Movies Extracted Columns**
The movie column had many columns that we did not need. We kept the following columns:
* budget - budget for film 
* genres - genre of film
* popularity - popularity score
* release_date - date that film was released
* revenue - revenue for film
* title - title of film

We dropped the following columns for our analysis because they were not relevant to the scope of our topic: 'id', 'imdb_id', 'original_language', 'overview', 'production_companies', 'production_countries', 'runtime', 'spoken_languages', 'status', 'tagline', 'vote_average', 'vote_count', 'production_companies_number', 'production_countries_number', 'spoken_languages_number', 'original_title'.

## Transform

**Cleaning Golden Globes DataFrame**

1. We kept only the categories that were about “Best Picture” of the year, leaving only the nominees and winner of the main movie categories each year. There were many other categories that were not "Best Picture" (e.g.,for specific actors/actresses, sound production) and defunct awards (e.g., International). One thing to note is that after the first several years, the general Best Picture category was expanded based on genre so we need to have an iterative string-selection process to keep the data that we need.  

2. We were interested in the removed the years that were released before or after our time frame (1997 to 2016).

3. We created a new column to contain "santized" film titles. Since it would be one of the columns we would merge on, we wanted to maximize our chances of consistency between the two data bases.
    * Titles were all converted to lowercase and any leading “The” was dropped (The Song of Bernadette --> song of bernadette). Several films also had a trailing “The” (ie “Robe, The”) which we removed as well.  These “The”s were stripped based on being the first four characters of the string or the last five characters of the string in order to avoid stripping any internal “The”s from any other movie titles.  We chose to get rid of any leading “The”s instead of moving a trailing “, The” to the front in case any movie that should have had a leading “The” didn’t for safety.
    * Similar process was deployed for ", A" and ", An".
    *  We also cleared the data for periods ("."), spaces (" "), and dashes ("-").
    * Column titles were also edited for consistency, converting them to lowercase and replacing spaces with underscores as needed.

**Cleaning Movie DataFrame**

1. We dropped the rows that did not have a film title. In this case there was only one row without a film title.

2. We extracted "release_year" from the "release_date", and removed the years that were released before or after our time frame (1997 to 2016).

3. We created a new column to contain "santized" film titles. Since it would be one of the columns we would merge on, we wanted to maximize our chances of consistency between the two data bases.
    * Titles were all converted to lowercase and any leading “The” was dropped. Several films also had a trailing “, The” which we removed as well.  
    * Similar process was deployed for ", A" and ", An".
    * We also cleared the data for periods ("."), spaces (" "), and dashes ("-").

4. We designated 0s in revenue and budget columns as NaNs so that we could add the "profit" column. The profit column returns "NaN" if either budget or revenue columns are "NaN".

**Merging Two DataFrames**

We use an interative merge strategy to combine the two data frames. We use two columns from each table for the merge: the sanitized film title column and release year column. We use two columns because there are film titles with multiple release dates (e.g., "Carmen"). 

* First Merge: We outer-merge the two data frames as whole, and extract 3 separate tables from this merge.
    1. Table of matched items, with all columns.
    2. Table of unmatched Golden Globe items with Golden Globe columns.
    3. Table of unmatched movies with movies columns.

* Second Merge: We outer-merge the two unmatched dataframes from first merge after performing transformations. We transform the Golden Globe items by checking if theres a minor discrpency in the "release year" and perform merge. We extract 3 separate tables from this merge.
    1. Table of matched items, with all columns.
    2. Table of unmatched Golden Globe items with Golden Globe columns.
    3. Table of unmatched movies with movies columns.

* Third (Final) Merge: We outer-merge the two unmatched dataframes from third merge after performing transformations. We perform individual transformations to match the two dataframes. We extract 3 separate tables from this merge.
    1. Table of **matched and unmatched** items, with all columns.
    2. For reference only: Table of unmatched Golden Globe items with Golden Globe columns.
    3. For reference only: Table of unmatched movies with movies columns.

* We append tables from our merges to have the full length of our dataframe.
    1. Table of matched items from first merge.
    2. Table of matched items from second merge.
    3. Table of matched and unmatched items from third merge (this contains movies that have not been nominated for a Golden Globe, and two Golden Globe nominated movies that were not in the movies database).

* Final step is to select our final columns for our database and clean the titles. We selected and revised the column titles to look as follows:
    * title (string) - title of film
    * genres (string) - genre of film
    * release_year (int) - year that film was released
    * nom_category (string) - nomination category (NaN if film was not nominated)
    * nom_year (int) - year film was nominated (NaN if film was not nominated)
    * win (boolean) - whether or not film won the nomination (NaN if film was not nominated)
    * popularity (float) - popularity score
    * budget (float) - budget for film (NaN if information is not available)
    * revenue (float) - revenue for film (NaN if information is not available)
    * profit (float) - profit for film (NaN if budget or revenue information is not available)

## Load
We created an engine to export our database to a sql database file. We thought it would be the best solution to loading our dataframe because:
* We were able to load the dataframe to a database without having to narrowly define the variety of datatypes we had in our columns.
* Although we inteded to make title and release_year our primary keys, we could not do so. We wanted to include the universe of all movies in our data sources between the years of 1997 to 2016 (hence the outer-merge). Unfortunately, the movies did not perfectly match to each other- there were two items from Golden Globes that did not match to the movies data base. We were able to load the data to this database without running into issues with primary keys. Our recommendation for those who use this data base is to utilize a serial primary key.