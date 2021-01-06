# Data modelling with Postgres

This repo contains my solution for the Data modelling with Postgres project of the [Udacity Data Engineering Nanodegree](https://www.udacity.com/course/data-engineer-nanodegree--nd027).

## Purpose

The project involves an imaginary music streaming startup called Sparkify. Sparkify is interested in analytical insights into the streaming habits of the users. Sparkify has song metadata as well as user activity logs, both as JSON files. Data is a subset of the [Million Song Dataset](http://millionsongdataset.com/) [1] We need to setup a database schema and an ETL pipeline to convert these file into a database that is usable for answering analytical queries.

## Database schema design

The database is setup using a star schema with a fact table containing song plays, and four related dimension tables.

### songplays table

The `songplays` fact table will allow us to query and aggregate the song play events we are interested in:

| Column                    | Data type |
| ------------------------- | --------- |
| songplay_id (primary key) | serial    |
| start_time                | timestamp |
| user_id                   | int       |
| level                     | varchar   |
| song_id                   | varchar   |
| artist_id                 | varchar   |
| session_id                | int       |
| location                  | varchar   |
| user_agent                | varchar   |

I'm using an auto incrementing integer (serial) as the songplay ID, the remaining columns are defined based on the available log data.

The additional tables are dimension tables providing descriptive text details of the songs, the artists, and the users referenced in the song plays, as well as a time dimension table to provide various time components (hour, day, weekday etc.)

### users table

User information referenced by user ID. Gender is M/F and can be stored as `char(1)`.

| Column                    | Data type |
| ------------------------- | --------- |
| user_id (primary key)     | int       |
| first_name                | varchar   |
| last_name                 | varchar   |
| gender                    | char(1)   |
| level                     | varchar   |

### songs table

Song metadata referenced by song ID. The artist is referenced through the `artist_id` column.

| Column                    | Data type |
| ------------------------- | --------- |
| song_id (primary key)     | varchar   |
| title                     | varchar   |
| artist_id                 | varchar   |
| year                      | smallint  |
| duration                  | decimal   |

### artists table

Artists metadata referenced by artist ID.

| Column                    | Data type |
| ------------------------- | --------- |
| artist_id (primary key)   | varchar   |
| name                      | varchar   |
| location                  | varchar   |
| latitude                  | decimal   |
| longitude                 | decimal   |

### time table

The time table allow aggregations by hour, day etc. The numbers are all on a limited scale, so I use the `smallint` type for minimal storage.

| Column                    | Data type |
| ------------------------- | --------- |
| start_time                | timestamp |
| hour                      | smallint  |
| day                       | smallint  |
| week                      | smallint  |
| month                     | smallint  |
| year                      | smallint  |
| weekday                   | smallint  |

## ETL pipeline

The ETL pipeline `etl.py` will populate the database tables using data from song files and log files:

### Song files

The song files are used to populate the songs table. The song files also contain artist information which is used to populate the artists table. Multiple songs may be from the same artist, but an `ON CONFLICT` handler in the SQL makes sure the artist is only inserted once.

## Log files

The log files contain song plays and other actions, but here we are only interested in song plays, so all other actions are filtered out.

From the timestamp, multiple time components (day, hour etc.) are extracted and stored in the time table.

The user information provided in the logs is used to populate the users table, and an `ON CONFLICT` handler ensures we only insert each user once.

Each song play event is inserted into the songplays table. Since artist and song metadata are stored in separate tables we use these tables to look up the song ID and artist ID, and these IDs are stored in the song play table.

## Queries

Please see `test.ipynb` for queries, including:

* Number of songs played by hour of the day: Seems to peek around 18 o'clock.
* Most popular songs: Unfortunately 'Setanta matins' is the only song in the logs we have metadata for.
* Number of plays per user for each subscription level: Paid users play 8x more songs than free users

## How to run

The code requires a Python environment to run, and the following packages installed:

* pandas
* psycopg2

Additionally, a PostgreSQL server must be running. A docker image configured with the correct credentials is available:

```
docker pull onekenken/postgres-student-image
docker run -d --name postgres-student-container -p 5432:5432 onekenken/postgres-student-image
```

Use Python to run the scripts:

```
# Setup database tables
python3 create_tables.py
# Run ETL pipeline
python3 etl.py
```

The repo also contains two notebooks which may be run using Jupyter Notebook.

## References

[1] Thierry Bertin-Mahieux, Daniel P.W. Ellis, Brian Whitman, and Paul Lamere. The Million Song Dataset. In Proceedings of the 12th International Society for Music Information Retrieval Conference (ISMIR 2011), 2011.