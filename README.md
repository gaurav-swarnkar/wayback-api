# Wayback day-data API

The Wayback app gets everything it shows about a day from this API. These
files are the API: any static host (a CDN, S3, GitHub, nginx) can serve them
as they are. Later you can replace them with your own data server, as long as
it returns the same JSON.

## Endpoints

All paths are relative to the base URL the app is configured with:
`DEFAULT_API_BASE_URL` in `src/lib/config.ts`, or the `EXPO_PUBLIC_API_URL`
environment variable at build time.

The app asks for the month file first, and falls back to the per-day v1 file for
months that don't have one yet.

### `GET /v2/months/{MM}.json` (preferred)

One file per month, with every day of that month. Example: `/v2/months/10.json`.

```json
{
  "month": 10,
  "name": "October",
  "days": {
    "10-03": {
      "names": ["German Unity Day"],
      "script": "The third of October is a day of coming together.\n\nIn 1990, at the stroke of midnight, ...",
      "events": [
        { "year": 1990, "text": "East and West Germany are reunified.", "category": "conflict", "wiki": "German_reunification" }
      ],
      "births": [ { "year": 1969, "text": "Gwen Stefani, American singer", "wiki": "Gwen_Stefani" } ],
      "deaths": [],
      "facts": [ { "text": "German Unity Day replaced 17 June as Germany's national day after reunification in 1990.", "wiki": "German_Unity_Day" } ]
    }
  }
}
```

A day has the same fields as the v1 day file below, plus:

| Field    | Type   | Meaning |
| -------- | ------ | ------- |
| `script` | string | The day told as a story (150 to 200 words). The narrator reads it aloud, pausing briefly between sentences and longer between paragraphs (separated by a blank line). The app adds the date line ("We are travelling back to October 3.") before it. Wayback is about the day of the year, so a script never refers to a "current" year, only to the years events happened. |
| `facts`  | object[] | Optional "Did you know?" notes: `{ "text", "wiki"?, "image"? }`. Shown after the day's names in the Did You Know group. |

Every story (event, birth or death) can also carry:

| Field   | Type   | Meaning |
| ------- | ------ | ------- |
| `image` | string | Direct HTTPS URL of the photo for the story's card and the page's rotating header. Use this when you host your own images. |
| `wiki`  | string | English Wikipedia article (e.g. `Sputnik_1`). When there is no `image`, the app shows that article's lead photo. |

Stories with neither show a coloured background instead.

### `GET /v1/days/{MM}-{DD}.json` (older, per day)

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
5. **Did You Know**: each entry in `names`, then each entry in `facts`

To give every group 3–5 cards, send 3–5 stories of each kind. `deaths` is part
of the format but isn't shown yet.

When a day has no data, respond with **404**. The app shows "No stories for
this day yet" instead of a connection error.

### `GET /v1/index.json`

Lists the days that have data: `{ "version": 1, "days": ["09-01", …] }`.

## Current coverage

- **v2 month files**: January, February (including 29 February), March, October, November and December. Every day has at least 4 Headlines, 3 Space, 3 Turning Points, 5 Birthdays and 3 Did You Know cards (29 January, 4, 19 and 29 February, and 12 March have 2 Space cards). Every day has a script, and every story has a `wiki` photo source and an explicit `category`. The source content is in `content/` in the app repo; `python3 content/build.py` regenerates these files.
- **v1 day files**: September and October.

All stories and scripts were written without checking against sources, so please review them before you rely on them.
