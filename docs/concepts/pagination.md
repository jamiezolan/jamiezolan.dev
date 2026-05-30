# Pagination

List endpoints return results in pages using **cursor-based** pagination. Cursors are stable under inserts and deletes, which offset-based paging is not — you won't skip or repeat records when the underlying data changes mid-iteration.

## Requesting a page

```bash
curl "https://api.pudgypay.dev/v1/charges?limit=25" \
  -H "Authorization: Bearer $PUDGY_KEY"
```

`limit` ranges from 1 to 100 and defaults to 25.

## The list response

```json
{
  "object": "list",
  "data": [ { "id": "ch_3Nf...", "object": "charge" }, "…" ],
  "has_more": true,
  "next_cursor": "ch_2Mp9xQ"
}
```

- **`data`** — the page of objects, newest first.
- **`has_more`** — whether another page exists.
- **`next_cursor`** — pass this as `starting_after` to fetch the next page.

## Iterating every page

```python
def iterate_charges():
    cursor = None
    while True:
        params = {"limit": 100}
        if cursor:
            params["starting_after"] = cursor
        page = get("/v1/charges", params=params).json()
        yield from page["data"]
        if not page["has_more"]:
            return
        cursor = page["next_cursor"]
```

!!! tip "Don't build your own cursors"
    Treat `next_cursor` as an opaque token. Its format is not part of the API contract and may change. Pass back exactly what you received.
