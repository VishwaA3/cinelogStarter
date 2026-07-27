# PR Response Doc: CineLog Watchlist Feature

## AI Usage

## Comment 1: Rename
**What I did:** Renamed `save_to_watchlist()` to `add_to_watchlist()` in `services/watchlist_service.py` to match the project's `verb_to_noun` naming convention used by `add_to_collection()`. Updated the import and call site in `routes/watchlist/watchlist.py` (the `add_film` route handler).
**How I verified:** Searched the codebase for all references to `save_to_watchlist` to confirm only one call site existed (`routes/watchlist/watchlist.py`). Ran the full test suite (`pytest tests/ -v`) after the change to confirm nothing broke.

## Comment 2: Deduplication
**What I did:** Added an `AlreadyInWatchlistError` exception class and a duplicate check inside `add_to_watchlist()`, following the same pattern as `add_to_collection()` in `collection_service.py`: after confirming the film exists, query for an existing `WatchlistEntry` with the same `user_id` and `film_id`, and raise `AlreadyInWatchlistError` if one is found, before creating a new entry.
**How I verified:** Ran the full test suite to confirm existing tests still pass.

## Comment 3: Missing test
**What I did:** Created `tests/test_watchlist.py` with `test_add_to_watchlist_nonexistent_film_raises`, mirroring `test_add_to_collection_nonexistent_film_raises` from `test_collection.py`. Used the same `app`, `sample_user`, and `sample_film` fixture pattern, and asserted that calling `add_to_watchlist()` with a nonexistent `film_id` raises `FilmNotFoundError`.
**How I verified:** Ran `pytest tests/test_watchlist.py -v` (passed), then the full suite `pytest tests/ -v` (all 5 tests passed).

## Comment 4: Default visibility
**My position:**
**Reasoning:**
**Tradeoff acknowledged:**

## Comment 5: Sort order
**My position:**
**Reasoning:**
**Engagement with reviewer's point:**

## Comment 6: Rebase
**What conflicted:** A conflict occurred in `models.py` on the `WatchlistEntry.film_id` column. My branch still defined it as `db.Integer` (with `index=True`), while `upstream/main` had migrated it to `db.String(36)` as part of the UUID refactor.
**How I resolved it:** Kept the UUID type (`db.String(36)`) from main to match the rest of the schema, while preserving the `index=True` attribute from my original definition. After resolving `models.py`, I audited `services/watchlist_service.py` and `tests/test_watchlist.py` for any remaining integer-ID assumptions: updated a stale docstring that still described `film_id` as an integer, and updated my test's fake film ID from an integer (`99999`) to a UUID-formatted string to match the new schema.

## PR Description