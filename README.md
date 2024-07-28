# Vector-SQL Database with MyScale

## Introduction

Vector databases are very popular nowadays. Mostly they are used to store the high-dimensional vector embeddings of unstructured data like text and images. They also allow us to perform semantic searches very efficiently. Long story short, they are better than traditional relational databases for this particular use case.

Recently, I came up with an interesting problem statement where I was given structured data meaning rows and columns containing both numerical as well as categorical features. I had to find a way to store them in a vector database and use it to increase query efficiency.

After doing my research, I came up with a solution to use a Vector-SQL Database. MyScale database is open-source and is built on another open-source relational database, ClickHouse. So, MyScale leverages the stability and reliability of ClickHouse, which itself uses columnar storage for faster querying but also gives an added advantage to store vector embeddings with the functionality to perform similarity search over them.

The best part about this is that we can do all these with SQL queries, allowing us to do more complex queries, unlocking endless possibilities. So technically speaking, we are getting the best of both worlds here.

I locally set up MyScale, and the results are phenomenal.

Here’s the GitHub repo for the MyScale DB: [MyScaleDB](https://github.com/myscale/MyScaleDB)

Here is my repo: [Vector-SQL-Database-Myscaledb](https://github.com/Harshitawake/Vector-SQL-Database-Myscaledb)

Feel free to look into this.

## SQL Commands for MyScale DB

### Create Table in MyScale DB
```sql
CREATE TABLE app_logs_all_1 (
    `appBetLogID` INT,
    `appDescription` TEXT,
    `appLogCode` INT,
    `appClientID` INT,
    `appBetID` INT,
    `appBetDetailID` FLOAT,
    `appIsBack` FLOAT,
    `appRate` FLOAT,
    `appStake` INT,
    `appPlaceBetInfo` TEXT,
    `appIsMasterLimit` FLOAT,
    `appCreatedBy` INT,
    `appDescription_embedding` Array(Float32),
    `appPlaceBetInfo_embedding` Array(Float32),
    CONSTRAINT check_length CHECK length(appDescription_embedding) = 384,
    CONSTRAINT check_length CHECK length(appPlaceBetInfo_embedding) = 384
)
ENGINE = MergeTree
ORDER BY appBetLogID;
