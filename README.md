# smallpond

[![CI](https://github.com/deepseek-ai/smallpond/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/deepseek-ai/smallpond/actions/workflows/ci.yml)
[![PyPI](https://img.shields.io/pypi/v/smallpond)](https://pypi.org/project/smallpond/)
[![Docs](https://img.shields.io/badge/docs-latest-brightgreen.svg)](https://deepseek-ai.github.io/smallpond/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## Project Overview

This repository is a fork of https://github.com/deepseek-ai/smallpond, enhanced with additional support for duckdb engine to aceess 3FS using hf3fs_usrbio.

## Using hf3fs_usrbio with DuckDB Engine

Ensure the 3FS storage cluster is mounted at /3fs/ , so duckdb engine  use hf3fs_usrbio to acess files in 3FS, and arrow engine use standard POSIX to acess files in 3FS.
```ptyhon
import smallpond

sp = smallpond.init(data_root="/3fs/duckdb/data_root/")
df = sp.read_parquet("/3fs/duckdb/prices.parquet")
df = df.repartition(3, hash_by="ticker")
df = sp.partial_sql("SELECT ticker, min(price), max(price) FROM {0} GROUP BY ticker", df)
print(df.to_pandas())
df.write_parquet("/3fs/duckdb/output/")
````

## Features

Smallpond is a lightweight data processing framework built on [DuckDB] and [3FS].

- 🚀 High-performance data processing powered by DuckDB
- 🌍 Scalable to handle PB-scale datasets
- 🛠️ Easy operations with no long-running services

## Installation

Python 3.8 to 3.12 is supported.

```bash
pip install smallpond
```

## Quick Start

```bash
# Download example data
wget https://duckdb.org/data/prices.parquet
```

```python
import smallpond

# Initialize session
sp = smallpond.init()

# Load data
df = sp.read_parquet("prices.parquet")

# Process data
df = df.repartition(3, hash_by="ticker")
df = sp.partial_sql("SELECT ticker, min(price), max(price) FROM {0} GROUP BY ticker", df)

# Save results
df.write_parquet("output/")
# Show results
print(df.to_pandas())
```

## Documentation

For detailed guides and API reference:
- [Getting Started](docs/source/getstarted.rst)
- [API Reference](docs/source/api.rst)

## Performance

We evaluated smallpond using the [GraySort benchmark] ([script]) on a cluster comprising 50 compute nodes and 25 storage nodes running [3FS].  The benchmark sorted 110.5TiB of data in 30 minutes and 14 seconds, achieving an average throughput of 3.66TiB/min.

Details can be found in [3FS - Gray Sort].

[DuckDB]: https://duckdb.org/
[3FS]: https://github.com/deepseek-ai/3FS
[GraySort benchmark]: https://sortbenchmark.org/
[script]: benchmarks/gray_sort_benchmark.py
[3FS - Gray Sort]: https://github.com/deepseek-ai/3FS?tab=readme-ov-file#2-graysort

## Development

```bash
pip install .[dev]

# run unit tests
pytest -v tests/test*.py

# build documentation
pip install .[docs]
cd docs
make html
python -m http.server --directory build/html
```

## License

This project is licensed under the [MIT License](LICENSE).
