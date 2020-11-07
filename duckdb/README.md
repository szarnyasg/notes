# DuckDB :duck:

## JDBC driver

```bash
rm -rf build && mkdir build && cd build/ && cmake -DJDBC_DRIVER=1 .. && GEN=ninja make; cd ..
```

```bash
cd tools/jdbc
cmake -DCMAKE_FIND_LIBRARY_PREFIXES=lib -DCMAKE_FIND_LIBRARY_SUFFIXES='.so;.a' .
```
Fails with:
```console
-- Configuring done
CMake Error: Cannot determine link language for target "duckdb_java".
CMake Error: CMake can not determine linker language for target: duckdb_java
```

## Testing load performance with the LDBC SNB data sets' comments:

```bash
python download-benchmark-data.py

DATA_DIR=sf0.1

# unzip comment CSV
gunzip ${DATA_DIR}/dynamic/comment_0_0.csv.gz

# copy
rm -f ldbc.duckdb
cat schema.sql | duckdb ldbc.duckdb
time echo "COPY comment FROM '${DATA_DIR}/dynamic/comment_0_0.csv' (DELIMITER '|', HEADER, TIMESTAMPFORMAT '%Y-%m-%dT%H:%M:%S.%g+00:00');" | duckdb ldbc.duckdb

# inserts
rm -f ldbc.duckdb
cat schema.sql | duckdb ldbc.duckdb
tail -n +2 ${DATA_DIR}/dynamic/comment_0_0.csv | awk -F '|' '{ print "INSERT INTO comment VALUES ('\''"substr($1, 1, 23)"'\'', "$2", '\''"$3"'\'', '\''"$4"'\'', '\''"gsub("'\''", "\\'\''", $5)"'\'', "$6", "$7", "$8", 0, 0); " }' > inserts.sql

# commit every x rows
lines=1
echo > insert-txs.sql
while IFS= read -r line
do
    echo "${line}" >> insert-txs.sql
    ((lines++ % 50000)) || echo "COMMIT; BEGIN TRANSACTION;" >> insert-txs.sql
done < inserts.sql

# check commit locations with:
# grep -n COMMIT insert-txs.sql

time { echo "BEGIN TRANSACTION;"; cat insert-txs.sql; echo "COMMIT;"; } | duckdb ldbc.duckdb
```
