# Modelling Record Stores Online

A small Python app that scrapes record-store catalogs, stores the releases in a local SQLite database, and lets you browse them through a REST API or from the command line.

Today it loads the [Hardwax](https://www.hardwax.com) homepage: artist, title, and track list for each release. The database is built so more stores can sit beside Hardwax. A Juno Records parser is sketched in, but Juno renders its catalog in the browser, so that parser currently returns no releases.

## What it does

1. **Scrape.** `parsers.py` fetches a store page and turns each listing into a release (artist, title, tracks). A release with no tracks is dropped.
2. **Validate.** `models.py` checks each release and gives the release and its tracks stable ids.
3. **Store.** `loader.py` writes artists, labels, formats, releases, and tracks into `hardwax.db`. The schema is in `DATA_MODEL_DESIGN.md`.
4. **Query.** `api.py` serves the database over HTTP. `queries.py` prints the same kinds of lookups in the terminal.

The database file is local and is not committed. Create it with the steps below before starting the API.

## Requirements

- Python 3.10 or newer
- Network access when you load live store data (`populate.py` requests hardwax.com)

## Setup

From the project directory:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

On Windows, activate the virtual environment with `.venv\Scripts\activate`.

## Run it

Create the empty database:

```bash
python build_database.py
```

That writes `hardwax.db` in the project directory.

Load the current Hardwax homepage into it:

```bash
python populate.py
```

Start the API (http://127.0.0.1:8000). Interactive docs are at http://127.0.0.1:8000/docs.

```bash
python api.py
```

Print sample lookups against the database:

```bash
python queries.py
```

Run the tests (they use an in-memory database and do not need `hardwax.db` or the network):

```bash
pytest tests.py -v
```

## API

`GET /` lists these routes. Most list endpoints accept an optional `store` query parameter (for example `?store=hardwax`).

| Method | Path | What it returns |
| --- | --- | --- |
| GET | `/stores` | Store names present in the database |
| GET | `/releases` | Releases, newest first |
| GET | `/release/{release_id}` | One release, including its tracks |
| GET | `/artist/{artist_name}` | Releases by that artist |
| GET | `/label/{label_name}` | Releases on that label |
| GET | `/format/{format_name}` | Releases in that format |
| GET | `/search?q=query` | Releases whose description contains the query |
| GET | `/stats` | Counts of releases, artists, labels, formats, and tracks, overall and per store |

Examples:

```bash
curl http://127.0.0.1:8000/releases?store=hardwax
curl http://127.0.0.1:8000/stats
curl "http://127.0.0.1:8000/search?q=dub"
```

## Project layout

| File | Role |
| --- | --- |
| `parsers.py` | Fetches store pages and extracts releases |
| `models.py` | Release and track models |
| `build_database.py` | Creates the SQLite schema |
| `loader.py` | Inserts releases into the database |
| `populate.py` | Scrapes Hardwax and loads the result |
| `db.py` | Opens and closes `hardwax.db` |
| `api.py` | FastAPI app |
| `queries.py` | Sample SQL printed to the terminal |
| `tests.py` | Query and model tests |
| `DATA_MODEL_DESIGN.md` | Table design |

## Current limits

The Hardwax scraper reads the public homepage, so you only get the releases listed there, not the full catalog. While loading, each release is stored with format `Digital`, and the label is saved as `Artist - Title` because those fields are not parsed from the page yet. Catalog number, price, and image URL columns exist in the schema and stay empty for scraped rows.
