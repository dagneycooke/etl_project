# ETL Project

## Team:
Dagney Cooke
Diana Silva
Heain Yee

## Data Sources
* [Golden Globe Awards](https://www.kaggle.com/unanimad/golden-globe-awards)
* [350 000+ movies](https://www.kaggle.com/stephanerappeneau/350-000-movies-from-themoviedborg#AllMoviesDetailsCleaned.csv)


## Details for the 3 phases of the project
### Extract

We found two data sources from Kaggle.  The first one was a CSV file of all Golden Globe nominees and winners from 1944 - 2020.  Our second source had multiple csv files related to 350,000 movies, and we chose to use their AllMoviesCleaned.csv file, which was pulled from themoviedb.org (it covers all movies until AUG17  - ~350k movies).  

Both of these files were read into pandas dataframes for cleaning, with the goal of being able to combine tables based on movie titles.


### Transform

**Golden Globes CSV File**
The first step to cleaning the Golden Globes dataframe was to remove columns we weren’t interested in, in this case information about the ceremony and the year the film was released.  We then dropped all categories that weren’t about the “Best Picture” of the year, leaving only the nominees and winner of the main movie categories each year.  These other categories ranged from Best Actor/Actress to defunct “International Cooperation”-type awards.    One thing to note is that after the first several years, the general Best Picture category was expanded based on genre so we couldn’t just select for one string.  
    
Once the extraneous info was removed, we focused on cleaning the column we wanted to merge on - film title.  Titles were all converted to lowercase and any leading “The” was dropped (The Song of Bernadette --> song of bernadette).  Several films also had a trailing “The” (ie “Robe, The”) which we removed as well.  These “The”s were stripped based on being the first four characters of the string or the last five characters of the string in order to avoid stripping any internal “The”s from any other movie titles.  We chose to get rid of any leading “The”s instead of moving a trailing “, The” to the front in case any movie that should have had a leading “The” didn’t for safety.  This will enable us to easily merge on name.

Column titles were also edited for consistency, converting them to lowercase and replacing spaces with underscores as needed.

**Movie CSV File**
First step for the cleaning data from the csv file was to drop the rows that did not have a film title. In this case there was only one row without a film title.

The next step was to drop the unnecessary columns from the source file (based on our desired table). These columns had information such as id from IMDB, language, overview and tagline of the movie, production, duration and votes.
    
After deleting the unnecessary columns and rows, we started working on formating some columns.  Following the same concept as the Golden Globes process for the column we wanted to base our merge on film title, so we also converted titles to lowercase and removed any leading “The”s.  We also parsed the year from the release date column to match the Golden Globes table, in order to have one more column to merge the 2 data frames. This became a need after we realized that there are several films with the same title (but with different release dates). For example, the title “Carmen” has 40 different release dates. 

To finalize the table, we decided to add a final column “Profit”, which would be a calculated column based on Revenue and Budget.  Before doing these calculations, we confirmed if the cells in both the Budget and Revenue columns had usable info. For some rows, both Budget and Revenue were 0, so we converted them to NaN values.  The resulting calculations for these rows was also NaN.

Like the Golden Globes table, column titles were edited for consistency, converting them to lowercase and replacing spaces with underscores as needed.

### Load

