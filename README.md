# MDS in a box
This project serves as end to end example of running the "Modern Data Stack" in a local environment. For those looking for a more integrated experience, devcontainers have been implemented as well. If you have docker and WSL installed, the container can booted up right from VS Code.

## Current progress
Right now, you can get the nba schedule and elo ratings from this project and generate the following query. more to come, see to-dos at bottom of readme. And of course, the dbt docs are self hosted in Github Pages, [check them out here](https://matsonj.github.io/nba-monte-carlo/).
![image](https://user-images.githubusercontent.com/16811433/195012880-adf8da03-ab16-4c16-8080-95514fb41c21.png)
![image](https://user-images.githubusercontent.com/16811433/195012951-dde884a0-88f5-48d5-8203-b6f06ba7dbd4.png)

## Getting started - Windows
1. Create your WSL environment. Open a PowerShell terminal running as an administrator and execute:
```
wsl --install
```
* If this was the first time WSL has been installed, restart your machine.

2. Open Ubuntu in your terminal and update your packages. 
```
sudo apt-get update
```
3. Install python3.
```
sudo apt-get install python3.8 python3-pip python3.8-venv
```
4. clone the this repo.
```
mkdir meltano-projects
cd meltano-projects
git clone https://github.com/matsonj/nba-monte-carlo.git
# Go one folder level down into the folder that git just created
cd nba-monte-carlo
```
5. build your project & run your pipeline
```
make build
make pipeline
```
6. Connect duckdb to superset. first, create an admin users
```
meltano invoke superset:create-admin
```
 - then boot up superset
```
meltano run superset:ui
```
 - lastly, connect it to duck db. navigate to localhost:8088, login, and add duckdb as a database.

   - SQL Alchemy URL: ```duckdb:////tmp/mdsbox.db```

   - Advanced Settings > SQL Lab > ✔ Expose Database in SQL Lab > ✔ Allow CREATE VIEW AS & ✔ Allow this database to be explored & ✔ Allow DML

   - Since the dbt run is materialized to `.parquet` files only, to make the tables available in Superset, go to SQL Lab > SQL Editor and run:
   
       ```c
       CREATE VIEW initialized_seeding as SELECT * FROM read_parquet('/tmp/data_catalog/conformed/initialize_seeding.parquet');
       CREATE VIEW playoff_sim_r1 as SELECT * FROM read_parquet('/tmp/data_catalog/conformed/playoff_sim_r1.parquet');
       CREATE VIEW playoff_sim_r2 as SELECT * FROM read_parquet('/tmp/data_catalog/conformed/playoff_sim_r2.parquet');
       CREATE VIEW playoff_sim_r3 as SELECT * FROM read_parquet('/tmp/data_catalog/conformed/playoff_sim_r3.parquet');
       CREATE VIEW playoff_sim_r4 as SELECT * FROM read_parquet('/tmp/data_catalog/conformed/playoff_sim_r4.parquet');
       CREATE VIEW reg_season_summary as SELECT * FROM read_parquet('/tmp/data_catalog/conformed/reg_season_summary.parquet');
       CREATE VIEW season_summary as SELECT * FROM read_parquet('/tmp/data_catalog/conformed/season_summary.parquet');
       CREATE VIEW reg_season_end as SELECT * FROM read_parquet('/tmp/data_catalog/conformed/reg_season_end.parquet');
       ```


7. Explore your data inside superset. Go to SQL Labs > SQL Editor and write a custom query. A good example is ```SELECT * FROM reg_season_end```.


## Using Parquet instead of a database
This project leverages parquet instead of a database for file storage. This is experimental and implementation will evolve over time.

## Todos
- [x] replace reg season schedule with 538 schedule
- [x] add table for results
- [x] add config options in dbt vars to ignore completed games
- [x] make simulator only sim incomplete games
- [x] add table for new ratings
- [x] add config to use original or new ratings
- [ ] cleanup dbt-osmosis
- [ ] clean up env vars + implement incremental builds
- [ ] clean up dev container plugins (remove irrelevant ones, add some others)
- [ ] add dbt tests on simulator tables that no numeric values are null (elo ratings, home team win probabilities)

## Optional stuff
- [x] add dbt tests
- [ ] add model descriptions
- [x] change elo calculation to a udf
- [x] make playoff elimination stuff a macro (param: schedule type)

## Source Data
The data contained within this project comes from [538](https://data.fivethirtyeight.com/#nba-forecasts), [basketball reference](https://basketballreference.com), and [draft kings](https://www.draftkings.com). 
