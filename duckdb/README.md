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

