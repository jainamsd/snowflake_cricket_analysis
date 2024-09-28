# Cricket Data Warehouse Project

## Overview

This project involves creating a data warehouse for managing and analyzing cricket match data. The architecture is designed to load, clean, and store cricket data efficiently using Snowflake.

## Table of Contents

- [Project Objectives](#project-objectives)
- [Schema Design](#schema-design)
- [Data Loading Process](#data-loading-process)
- [Key Learnings](#key-learnings)

## Project Objectives

- Build a data warehouse that captures essential cricket match information.
- Design fact and dimension tables for efficient querying and analysis.
- Automate the data loading process from local machines to Snowflake.
- Create dashboards for quick insights into cricket match statistics.

## Schema Design

### Dimension Tables

1. **Date Dimension (`date_dim`)**
   ```sql
   CREATE OR REPLACE TABLE date_dim (
       date_id INT PRIMARY KEY AUTOINCREMENT, -- Unique identifier for each date
       full_dt DATE,                          -- Full date
       day INT,                               -- Day of the month
       month INT,                             -- Month of the year
       year INT,                              -- Year
       quarter INT,                           -- Quarter of the year
       dayofweek INT,                        -- Day of the week (1=Monday, 7=Sunday)
       dayofmonth INT,                       -- Day of the month
       dayofyear INT,                        -- Day of the year
       dayofweekname VARCHAR(3),            -- Short name of the day (e.g., Mon)
       isweekend BOOLEAN                     -- Indicator if it's a weekend
   );
``

2. **Referee Dimension (`referee_dim`)**
```sql
CREATE OR REPLACE TABLE referee_dim (
    referee_id INT PRIMARY KEY AUTOINCREMENT, -- Unique identifier for each referee
    referee_name TEXT NOT NULL,                -- Name of the referee
    referee_type TEXT NOT NULL                 -- Type of referee (e.g., on-field, third umpire)
);
```
3. **Team Dimension (`team_dim`)**
```sql
CREATE OR REPLACE TABLE team_dim (
    team_id INT PRIMARY KEY AUTOINCREMENT, -- Unique identifier for each team
    team_name TEXT NOT NULL                  -- Name of the team
);
```
4. **Player Dimension (`player_dim`)**
```sql
CREATE OR REPLACE TABLE player_dim (
    player_id INT PRIMARY KEY AUTOINCREMENT,  -- Unique identifier for each player
    team_id INT NOT NULL,                     -- Foreign key referencing the team
    player_name TEXT NOT NULL,                -- Name of the player
    CONSTRAINT fk_team_player_id FOREIGN KEY (team_id) REFERENCES team_dim (team_id)
);
```
5. **Venue Dimension (`venue_dim`)**
```sql
CREATE OR REPLACE TABLE venue_dim (
    venue_id INT PRIMARY KEY AUTOINCREMENT,   -- Unique identifier for each venue
    venue_name TEXT NOT NULL,                  -- Name of the venue
    city TEXT NOT NULL,                        -- City where the venue is located
    state TEXT,                               -- State where the venue is located
    country TEXT,                             -- Country where the venue is located
    continent TEXT,                           -- Continent where the venue is located
    end_Names TEXT,                          -- Names of the ends of the pitch
    capacity NUMBER,                         -- Seating capacity of the venue
    pitch TEXT,                              -- Description of the pitch
    flood_light BOOLEAN,                     -- Indicator if the venue has floodlights
    established_dt DATE,                     -- Date the venue was established
    playing_area TEXT,                       -- Description of the playing area
    other_sports TEXT,                       -- Other sports played at the venue
    curator TEXT,                            -- Name of the curator
    latitude NUMBER(10,6),                  -- Latitude of the venue location
    longitude NUMBER(10,6)                  -- Longitude of the venue location
);
```
6. **Match Type Dimension (`match_type_dim`)**
```sql
CREATE OR REPLACE TABLE match_type_dim (
    match_type_id INT PRIMARY KEY AUTOINCREMENT, -- Unique identifier for each match type
    match_type TEXT NOT NULL                       -- Type of the match (e.g., ODI, Test)
);
```
### Fact Table
 ** Match fact Table(`Match fact table`)**
```sql
CREATE OR REPLACE TABLE match_fact (
    match_id INT PRIMARY KEY,                   -- Unique identifier for each match
    date_id INT NOT NULL,                       -- Foreign key referencing the date dimension
    referee_id INT NOT NULL,                    -- Foreign key referencing the referee dimension
    team_a_id INT NOT NULL,                     -- Foreign key referencing team A
    team_b_id INT NOT NULL,                     -- Foreign key referencing team B
    match_type_id INT NOT NULL,                 -- Foreign key referencing the match type
    venue_id INT NOT NULL,                      -- Foreign key referencing the venue
    total_overs NUMBER(3),                     -- Total overs for the match
    balls_per_over NUMBER(1),                   -- Balls per over (usually 6)
    overs_played_by_team_a NUMBER(2),          -- Overs played by team A
    bowls_played_by_team_a NUMBER(3),          -- Balls bowled by team A
    extra_bowls_played_by_team_a NUMBER(3),    -- Extra balls played by team A
    extra_runs_scored_by_team_a NUMBER(3),      -- Extra runs scored by team A
    fours_by_team_a NUMBER(3),                  -- Fours hit by team A
    sixes_by_team_a NUMBER(3),                  -- Sixes hit by team A
    total_score_by_team_a NUMBER(3),            -- Total score by team A
    wicket_lost_by_team_a NUMBER(2),            -- Wickets lost by team A
    overs_played_by_team_b NUMBER(2),          -- Overs played by team B
    bowls_played_by_team_b NUMBER(3),          -- Balls bowled by team B
    extra_bowls_played_by_team_b NUMBER(3),    -- Extra balls played by team B
    extra_runs_scored_by_team_b NUMBER(3),      -- Extra runs scored by team B
    fours_by_team_b NUMBER(3),                  -- Fours hit by team B
    sixes_by_team_b NUMBER(3),                  -- Sixes hit by team B
    total_score_by_team_b NUMBER(3),            -- Total score by team B
    wicket_lost_by_team_b NUMBER(2),            -- Wickets lost by team B
    toss_winner_team_id INT NOT NULL,           -- Foreign key for the toss winner
    toss_decision TEXT NOT NULL,                 -- Decision made after the toss
    match_result TEXT NOT NULL,                  -- Result of the match
    winner_team_id INT NOT NULL,                 -- Foreign key for the winning team
    CONSTRAINT fk_date FOREIGN KEY (date_id) REFERENCES date_dim (date_id),
    CONSTRAINT fk_referee FOREIGN KEY (referee_id) REFERENCES referee_dim (referee_id),
    CONSTRAINT fk_team1 FOREIGN KEY (team_a_id) REFERENCES team_dim (team_id),
    CONSTRAINT fk_team2 FOREIGN KEY (team_b_id) REFERENCES team_dim (team_id),
    CONSTRAINT fk_match_type FOREIGN KEY (match_type_id) REFERENCES match_type_dim (match_type_id),
    CONSTRAINT fk_venue FOREIGN KEY (venue_id) REFERENCES venue_dim (venue_id),
    CONSTRAINT fk_toss_winner_team FOREIGN KEY (toss_winner_team_id) REFERENCES team_dim (team_id),
    CONSTRAINT fk_winner_team FOREIGN KEY (winner_team_id) REFERENCES team_dim (team_id)
);

```

** Delivery fact Table(`Delivery fact table`)**
```sql
CREATE OR REPLACE TABLE delivery_fact (
    match_id INT,                                -- Foreign key referencing the match fact table
    team_id INT,                                 -- Foreign key referencing the team dimension
    bowler_id INT,                               -- Foreign key referencing the bowler
    batter_id INT,                               -- Foreign key referencing the batter
    non_striker_id INT,                          -- Foreign key referencing the non-striker
    over INT,                                    -- Over number
    runs INT,                                    -- Runs scored in the delivery
    extra_runs INT,                              -- Extra runs (if any)
    extra_type VARCHAR(255),                     -- Type of extra (e.g., no ball)
    player_out VARCHAR(255),                     -- Player who got out (if any)
    player_out_kind VARCHAR(255),                -- Type of dismissal
    CONSTRAINT fk_del_match_id FOREIGN KEY (match_id) REFERENCES match_fact (match_id),
    CONSTRAINT fk_del_team FOREIGN KEY (team_id) REFERENCES team_dim (team_id),
    CONSTRAINT fk_bowler FOREIGN KEY (bowler_id) REFERENCES player_dim (player_id),
    CONSTRAINT fk_batter FOREIGN KEY (batter_id) REFERENCES player_dim (player_id),
    CONSTRAINT fk_striker FOREIGN KEY (non_striker_id) REFERENCES player_dim (player_id)
);
```

### Data loading process
1. Loading Data into Snowflake

Data was loaded from local machines to Snowflake stages using the following command:
```sql
COPY INTO @my_stage FROM '/path/to/local/files' FILE_FORMAT = (TYPE = 'JSON');

```
2. Loading Delta Datasets

Delta datasets were loaded using the same COPY INTO command, ensuring that only new or modified data was uploaded.

3. Querying Stage Files

Used the $ notation to query stage files, allowing for flexible access to data:
    ``` sql
     SELECT * FROM @my_stage/my_file.json;
     ```

4. Running Copy Commands for JSON Files
   - Data from JSON files was ingested into the tables using:
     ```sql
     
     COPY INTO match_fact FROM @my_stage/match_data.json FILE_FORMAT = (TYPE = 'JSON');
     ```

## Key Learnings

- **Understanding Data Warehousing Concepts**: Gained insights into the architecture of data warehouses, including fact and dimension tables.
- **Snowflake Functionality**: Learned to effectively utilize Snowflake for data storage, management, and querying.
- **Automation**: Developed automation strategies to streamline data loading processes, ensuring efficient data flow.
- **Dashboard Creation**: Explored visualization techniques to create dashboards that provide valuable insights into cricket match statistics.

## Getting Started

### Prerequisites
- Snowflake account
- Local environment set up for SQL
- JSON data files for cricket matches

### Set up the Snowflake environment:

- Ensure your Snowflake account is configured with the necessary permissions to create databases and tables.
- Load the data into Snowflake using the provided SQL scripts.

### Usage
- Modify the SQL scripts as necessary to adapt to your specific data sources or structures.
- Execute queries against the tables to analyze cricket match data.


