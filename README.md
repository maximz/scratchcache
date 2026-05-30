# scratchcache

`scratchcache` provides a small local read-through cache for files that are
expensive to read directly, such as files stored on a network drive.

The package currently exposes one helper, `local_machine_cache`, which takes a
source file path and a local cache directory. It returns a path to a local copy
that can be passed to normal file-reading code.

## Why It Exists

Some libraries repeatedly read the same input file and perform poorly when that
file lives on slower remote storage. `scratchcache` lets callers keep their
source-of-truth path unchanged while reading from a machine-local copy whenever
the cached copy is current.

The cache is for reads only. Do not write through the returned path.

## How It Works

`local_machine_cache(fname, local_machine_cache_dir)`:

- verifies that `fname` exists, otherwise raises `FileNotFoundError`
- hashes the source path to choose a stable cache filename
- preserves the source file's full extension chain, such as `.tar.gz`
- creates the cache directory if needed
- copies the source file with `shutil.copy2` when the cache file is missing or
  its modification time differs from the source
- returns the cached local `pathlib.Path`

Freshness is based on exact modification-time equality between the source file
and the cached copy.

## Installation

```bash
pip install scratchcache
```

The package supports Python 3.8 and newer and has no runtime dependencies.

## Usage

```python
from pathlib import Path

from scratchcache import local_machine_cache

source = Path("/Volumes/shared/data/example.csv")
cache_dir = Path("/tmp/scratchcache")

local_path = local_machine_cache(source, cache_dir)

with local_path.open() as f:
    rows = f.readlines()
```

Use the returned path for reading. If the original file changes later and its
modification time no longer matches the cached copy, the next call refreshes the
cache.

## Development

```bash
pip install -r requirements_dev.txt
pip install -e .
make test
make lint
make docs
```

The test suite is configured with `pytest`; docs are built with Sphinx. Releases
are versioned in both `setup.py` and `scratchcache/__init__.py`.
