# Saint Petersburg Buildings: CSV to SQL Analysis

[Русская версия](README.ru.md)

Final Java course project (variant 1). The program reads a CSV dataset of Saint Petersburg buildings, loads it into an SQLite database and answers three questions with SQL queries. The result of the first task is also plotted as a bar chart with JFreeChart.

## How it works

1. `Main` calls `SpbBaseParser.spbParser()`, which reads `base.csv` with Apache Commons CSV and creates one `SpbBase` object for each row of the table. The objects are collected into a list.
2. `SpbSql` opens a JDBC connection to the SQLite database `identifier.sqlite`. `SpbSql.addProduct()` inserts every parsed row into the table `table_name`.
3. The three tasks are then run one by one as SQL queries.

The repository already contains the filled database `identifier.sqlite`, so the insert loop in `Main` is commented out.

## Tasks

**1) Number of buildings by floor count.** The task uses the `number_of_floor` column. Its values are written as words such as "1-этажный" ("1-storey"), so to parse the number of floors correctly the query removes all characters that are not digits (the Russian letters and `-`) with `TRIM` and casts the rest to `INTEGER`. The program counts the buildings for every floor count from 1 to 28 and draws a bar chart from this data with `JFreeChart` (saved as `task1.png`).

```sql
SELECT COUNT(number)
FROM table_name
WHERE CAST(TRIM(number_floor, 'Мяюэьыъщшчцхфутсрпонмлкйизжёедгвба-') AS INTEGER) == ?
```

**2) Registered plots with prefix code 9881.** The query selects rows with `prefix_code = 9881` and `appellation = 'Зарегистрированный участок'` ("registered plot"). The `address` column is not used in the filter, because all buildings with `prefix_code = 9881` are already on Shlisselburgskoye Highway. The addresses for the output are collected in a `StringBuilder`.

```sql
SELECT address
FROM table_name
WHERE CAST(prefix_code AS INTEGER) == 9881 AND appellation == 'Зарегистрированный участок'
```

**3) Universities higher than 5 floors with a known construction year.** The query uses the columns `appellation = 'Университет'` ("university"), `number_of_floor > 5`, `prefix_code` and a non-empty `year_construction`. The `number_of_floor` column is parsed the same way as in the first task. While the addresses are added to the `StringBuilder`, the program computes the average `prefix_code` and appends it at the end.

```sql
SELECT address, prefix_code
FROM table_name
WHERE appellation LIKE 'Университет'
  AND CAST(TRIM(number_floor, 'Мяюэьыъщшчцхфутсрпонмлкйизжёедгвба-') AS INTEGER) > 5
  AND year_construction != ''
```

## Results

The program prints its output in Russian.

### Task 1: console output

![Task 1 console output](Первая%20задача.png)

### Task 1: chart

![Task 1 chart: number of buildings by floor count](task1.png)

### Task 2: console output

![Task 2 console output](Вторая%20задача.png)

### Task 3: console output

![Task 3 console output](Третья%20задача.png)

## How to run

Requirements: JDK 17 and Maven. Dependencies are listed in `pom.xml`: Apache Commons CSV, SQLite JDBC and JFreeChart.

Run the program from the repository root, because `base.csv` and `identifier.sqlite` are opened by relative paths:

```bash
mvn compile exec:java -Dexec.mainClass=com.betrayalove.Main
```

You can also open the project in IntelliJ IDEA as a Maven project and run `com.betrayalove.Main`.

## Project structure

| File | Purpose |
| --- | --- |
| `src/main/java/com/betrayalove/Main.java` | Entry point: runs the tasks and builds the chart |
| `src/main/java/com/betrayalove/SpbBaseParser.java` | Parses `base.csv` into `SpbBase` objects |
| `src/main/java/com/betrayalove/SpbBase.java` | One building record |
| `src/main/java/com/betrayalove/SpbSql.java` | SQLite connection, insert and the three task queries |
| `base.csv` | Source dataset |
| `identifier.sqlite` | SQLite database with the loaded data |
