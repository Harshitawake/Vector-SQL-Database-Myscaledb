# Vector-SQL Database with MyScale

## Introduction

Vector databases are very popular nowadays. Mostly they are used to store the high-dimensional vector embeddings of unstructured data like text and images. They also allow us to perform semantic searches very efficiently. Long story short, they are better than traditional relational databases for this particular use case.

Recently, I came up with an interesting problem statement where I was given structured data meaning rows and columns containing both numerical as well as categorical features. I had to find a way to store them in a vector database and use it to increase query efficiency.

After doing my research, I came up with a solution to use a Vector-SQL Database. MyScale database is open-source and is built on another open-source relational database, ClickHouse. So, MyScale leverages the stability and reliability of ClickHouse, which itself uses columnar storage for faster querying but also gives an added advantage to store vector embeddings with the functionality to perform similarity search over them.

The best part about this is that we can do all these with SQL queries, allowing us to do more complex queries, unlocking endless possibilities. So technically speaking, we are getting the best of both worlds here.
![myscale db architecture](Screen_shots/myscaledb.jpg)


I locally set up MyScale, and the results are phenomenal.

Here’s the GitHub repo for the MyScale DB: [MyScaleDB](https://github.com/myscale/MyScaleDB)

Here is my repo: [Vector-SQL-Database-Myscaledb](https://github.com/Harshitawake/Vector-SQL-Database-Myscaledb)

Feel free to look into this.


# Create Table in Myscale db

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
```

# Insert rows to the Table Created from the csv file saved in local

```sql
INSERT INTO app_logs_all_1 
SELECT
    toInt32(appBetLogID),
    appDescription,
    toInt32(appLogCode),
    toInt32(appClientID),
    toInt32(appBetID),
    toFloat32(appBetDetailID),
    toFloat32(appIsBack),
    toFloat32(appRate),
    toInt32(appStake),
    appPlaceBetInfo,
    toFloat32(appIsMasterLimit),

    toInt32(appCreatedBy),
    appDescription_embedding,
    appPlaceBetInfo_embedding

FROM file('/var/lib/clickhouse/user_files/outputfull1.csv', 'CSVWithNames');
```

# Create Vector index on embeddings columns

```sql
ALTER TABLE app_logs_all_1 ADD VECTOR INDEX vec_idx appDescription_embedding TYPE SCANN('metric_type=Cosine');
```

# Create another table to store the query

```sql
CREATE TABLE question_embeddings (
    id Int32,
    question String,
    embedding Array(Float32),
    CONSTRAINT check_length CHECK length(embedding) = 384
)
ENGINE = MergeTree()
ORDER BY id;
```

# Insert the queries from csv file into the table

```sql
INSERT INTO question_embeddings
SELECT 
    id,
    question,
    embedding
FROM file('/var/lib/clickhouse/user_files/question.csv', 'CSVWithNames');
```

# Query command on your table

```sql
SELECT appBetLogID,
    appLogCode,
    appClientID,
    appBetID,
    appBetDetailID,
    appIsBack,
    appRate,
    appStake,
    appIsMasterLimit 
FROM app_logs_all_1 
LIMIT 100;
```

# Query command on your table with similarity search and some condition

```sql
SELECT appBetLogID,
    appLogCode,
    appClientID,
    appBetID,
    appBetDetailID,
    appIsBack,
    appRate,
    appStake,
    appIsMasterLimit,
    distance(appDescription_embedding, (
        SELECT embedding
        FROM question_embeddings
        WHERE id = 1
    )) as distance
FROM app_logs_all_1
WHERE distance > 0.5 AND appBetLogID IN [308,309,52]
ORDER BY distance ASC
LIMIT 100;
```

# Query command on your table with similarity search

```sql
SELECT appBetLogID,
    appLogCode,
    appClientID,
    appBetID,
    appBetDetailID,
    appIsBack,
    appRate,
    appStake,
    appIsMasterLimit,
    distance(appDescription_embedding, (
        SELECT embedding
        FROM question_embeddings
        WHERE id = 1
    )) as distance
FROM app_logs_all_1
ORDER BY distance ASC
LIMIT 100;
```

# Query to find the closest description

```sql
SELECT appDescription
FROM app_logs_all_1
WHERE appLogCode = 1008
ORDER BY distance(appDescription_embedding, (
    SELECT embedding
    FROM question_embeddings
    WHERE id = 1
))
LIMIT 1;
```

# Query to find the closest description with distance

```sql
SELECT appDescription, distance(appDescription_embedding, (
    SELECT embedding
    FROM question_embeddings
    WHERE id = 1
)) as distance
FROM app_logs_all_1
ORDER BY distance ASC
LIMIT 1;
```
