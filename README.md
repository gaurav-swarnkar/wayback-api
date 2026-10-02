# Wayback day-data API

The Wayback app gets everything it shows about a day from this API. These
files are the API: any static host (a CDN, S3, GitHub, nginx) can serve them
as they are. Later you can replace them with your own data server, as long as
it returns the same JSON.

## Endpoints

All paths are relative to the base URL the app is configured with:
`DEFAULT_API_BASE_URL` in `src/lib/config.ts`, or the `EXPO_PUBLIC_API_URL`
environment variable at build time.

### `GET /v1/days/{MM}-{DD}.json`

Everything for one calendar day, across all years. Example: `/v1/days/10-03.json`.

```json
{
  "date": "10-03",
  "names": ["German Unity Day"],
  "events": [
    { "year": 1990, "text": "East and West Germany are reunified." }
  ],
  "births": [
    { "year": 1969, "text": "Gwen Stefani, American singer" }
  ],
  "deaths": []
}
```

| Field    | Type     | Meaning                                                   |
| -------- | -------- | --------------------------------------------------------- |
| `date`   | string   | `MM-DD` of the day                                        |
| `names`  | string[] | What the day is known as (observances, national days)    |
| `events` | Story[]  | Things that happened on this day                          |
| `births` | Story[]  | People born on this day                                   |
| `deaths` | Story[]  | People who died on this day                               |

A **Story** is:

| Field   | Type   | Required | Meaning                                         |
| ------- | ------ | -------- | ----------------------------------------------- |
| `year`  | number | yes      | Year it happened (negative for BC)              |
| `text`  | string | yes      | One-sentence description                        |
| `title` | string | no       | Short subject, used for "Read about …"          |
| `url`   | string | no       | Link opened when the story is tapped            |
| `image` | string | no       | HTTPS image URL; shown on cards and the hero    |
| `category` | string | no    | Events only: `news`, `science` or `conflict`. Picks the card group; if left out, the app guesses from the text |

### How the app shows a day

The details page is one swipeable row of cards, one story per card, grouped in
this order (at most 5 cards per group):

1. **Headlines**: `events` with category `news`
2. **Birthdays**: `births`
3. **Space**: `events` with category `science`
4. **Turning Points**: `events` with category `conflict`
5. **Did You Know**: each entry in `names`

To give every group 3–5 cards, send 3–5 stories of each kind. `deaths` is part
of the format but isn't shown yet.

When a day has no data, respond with **404**. The app shows "No stories for
this day yet" instead of a connection error.

### `GET /v1/index.json`

Lists the days that have data: `{ "version": 1, "days": ["09-01", …] }`.

## Current coverage

Every day in September and October: 633 stories in total (152 headlines, 125
space/science, 181 turning points, 182 birthdays) plus what each day is known
as. Every event has an explicit `category`. Many days still have only 1–2
Space or Headlines stories, and these stories were written without checking
sources, so please review them before you rely on them.
