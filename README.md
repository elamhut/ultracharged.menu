# Ultracharged Omarchy Menu

A maintained fork of [dzhibas/omarchy.dzhibas.menu](https://github.com/dzhibas/omarchy.dzhibas.menu)
(the Supercharged Omarchy menu), with fixes for current Omarchy releases.

A fork of the Omarchy command menu that answers questions instead of only
finding commands.

The stock menu is a launcher: you type, it filters, you pick. Everything below
is what this fork adds on top — searches that come back with an **answer** at
the head of the list. Enter copies it, or acts on it. Everything the stock menu
does (apps, themes, system commands, all of `omarchy-menu.jsonc`) still works
exactly as before.

## Features

Three prefixes say what kind of search you want; everything else is answered
as you type.

| Prefix | Does | Example |
| --- | --- | --- |
| `!` | Searches a site ([bangs](#bangs)) | `!y lofi beats` |
| `?` | Finds a file ([files](#find-files)) | `?invoice` |
| `$` | Quotes a stock ([quotes](#stock-quotes)) | `$aapl` |

### Open links and search the web

| Type | Get |
| --- | --- |
| `github.com/dzhibas` | Opens it in your browser |
| `https://claude.ai/code` | Opens it |
| `localhost:3000` | Opens it |
| *anything with no matches* | **Search the web** — no more dead-end "No matches" |

<img width="610" height="278" alt="image" src="https://github.com/user-attachments/assets/0337630e-cdc9-4c18-92b7-da767598ead9" />

Bare domains are recognised only for known top-level domains, so `MenuModel.js`
and `install.sh` stay ordinary searches. Links open with
`omarchy-launch-browser`, which resolves your default browser and focuses the
window. Web search goes to DuckDuckGo — change `webSearchTemplate` in
`Menu.qml` for anything else.

### Bangs

DuckDuckGo-style prefixes send a search straight to a site.

| Type | Get |
| --- | --- |
| `!y lofi beats` | Searches YouTube |
| `!yh weather` | Searches Yahoo |
| `!g omarchy` | Searches Google |
| `!ddg omarchy` | Searches DuckDuckGo |
| `!a usb-c hub` | Searches Amazon |
| `!ml iphone 13` | Searches Mercado Livre |
| `! omarchy` | Searches with the default engine |
| `!y` | Opens YouTube itself |

A bang takes over the list: nothing in the menu competes with it. Bangs are
case-insensitive and must come first. An unknown bang (`!w`) stays an ordinary
search, so the DuckDuckGo fallback can still resolve it with its own bang list.
Adding a bang is one line in the `BANGS` table in `MenuModel.js`.

### Find files

| Type | Get |
| --- | --- |
| `?invoice` | Files and folders under your home whose name contains `invoice` |
| `? menu.qml` | The space after `?` is optional |
| `?` | Your favourite files, ready to open |

Enter opens the file with its default application (`xdg-open`), or a folder in
your file manager. Exact names come first, then names that start with what you
typed, then the rest, with shallower paths ahead of deeper ones.

**Favourites.** Every file you open from here is remembered, ranked by
*frecency*: each open adds to its score, and the score halves for every two
weeks it goes unopened. The files you use now rise, and last month's project
sinks on its own. A bare `?` lists the top eight. While you type, matching
favourites (marked 󰓎) lead the list straight away, before the disk search
finishes. Files that no longer exist are dropped. The list is kept at
`~/.local/state/omarchy/menu-file-history.json`; delete it to start over.

The search is [`fd`](https://github.com/sharkdp/fd), which ships with Omarchy.
It matches names, not contents, ignores case, and, as fd does by default,
skips hidden and gitignored files, so caches and build output don't bury what
you are after. fd runs once per name you type, never two at a time, and the
previous results stay narrowed on screen while the next search runs.

### Stock quotes

| Type | Get |
| --- | --- |
| `$auvp11` | `150.49 BRL  ▼ 1.27 (0.84%)` · `AUVP11.SA · BTGTEVA AUVPCI · São Paulo · 16:19` |
| `$aapl` | `347.38 USD  ▲ 3.39 (0.99%)` |
| `$btc-usd` | Crypto pairs work too |
| `$^bvsp` | And indices |

Enter opens the quote on Yahoo Finance. Type the ticker the way you know it:
Yahoo's search finds the listing (`AUVP11` trades as `AUVP11.SA`). The change
is against the previous close, and the time is when the price was set, in the
exchange's own time zone.

Quotes come from Yahoo Finance's public endpoints, with no API key, and only
once you type a `$` ticker. One fetch runs at a time and a quote is reused for a
minute, so typing a ticker letter by letter doesn't fire a request per letter.
Nothing is found, or you're offline? Enter searches Yahoo Finance instead. A
ticker starts with a letter, which is what keeps `$120 to eur` a currency
conversion.

### Unit conversion

| Type | Get |
| --- | --- |
| `100 km to miles` | `62.1371 mi` |
| `20 c to f` | `68 °F` |
| `5 gb to mb` | `5000 MB` |
| `1 tb to gib` | `931.323 GiB` |
| `70 kg to lb` | `154.324 lb` |
| `2 cup to ml` | `473.176 ml` |

Length, mass, volume, temperature, data, speed, time and area. Data covers both
the decimal (`kb mb gb tb`) and binary (`kib mib gib tib`) ladders, because the
difference between them is most of why anyone asks. Units take their full names
too — `miles`, `pounds`, `celsius`, `kilometres`. No network, no cache.

<img width="612" height="282" alt="image" src="https://github.com/user-attachments/assets/f60d7032-64c6-4937-9747-5b89fad21d59" />

### Time zones

| Type | Get |
| --- | --- |
| `time in tokyo` | `18:13` · `Asia/Tokyo · 6 hours ahead` |
| `now in utc` | `09:13` · `UTC · 3 hours behind` |
| `time in pst` | `02:13` · `America/Los_Angeles · 10 hours behind` |

Any city the system knows — the zone list comes from `timedatectl` — plus the
usual abbreviations (`utc`, `cet`, `est`, `pst`, `ist`, `jst`, `aest`). The day
is shown alongside the time when it isn't today's.

<img width="616" height="281" alt="image" src="https://github.com/user-attachments/assets/29ff9d9f-d45c-40e3-9924-8f449788ea0f" />

### Developer utilities

| Type | Get |
| --- | --- |
| `uuid` | A random UUID v4 |
| `password 24` | A 24-character password |
| `sha256 omarchy` | `382a803dcc0de9…` |
| `base64 hello world` | `aGVsbG8gd29ybGQ=` |
| `b64d aGVsbG8=` | `hello` |
| `urlencode a b&c` | `a%20b%26c` |
| `urldecode a%20b` | `a b` |
| `epoch` | Current epoch seconds |
| `epoch 1787875351` | `Fri 28 Aug 2026 03:02:31` |

Keywords: `uuid` `password` `sha256` `base64` `b64` `b64d` `unbase64`
`urlencode` `urldecode` `epoch`.

Random bytes come from `/dev/urandom`, not `Math.random`, so `password` is
worth trusting. The answer holds still while you finish typing and is new the
next time you open the menu.

<img width="607" height="654" alt="image" src="https://github.com/user-attachments/assets/78eb2f0b-4b61-48d3-9a3b-1ae759c5ac09" />


### Kill a process

| Type | Get |
| --- | --- |
| `kill chromium` | The matching processes, with pid, CPU and memory |

Enter sends `SIGTERM`. Nothing runs `ps` until `kill` is typed, and the listing
is reused for a few seconds rather than re-run on every keystroke.

<img width="612" height="285" alt="image" src="https://github.com/user-attachments/assets/9d0e490d-c74d-466c-82a9-f0d05adf9f27" />

### Search synonyms

| Type | Get |
| --- | --- |
| `restart` | **Reboot** |
| `turn off` | **Shutdown** |

Some menu entries answer to the words people actually reach for. Synonyms only
find an entry: unlike `aliases` in `omarchy-menu.jsonc`, they are not
`omarchy menu summon` routes, so they cannot clash with one (`restart` is
already Update's). Add more in the `SEARCH_SYNONYMS` table in `MenuModel.js`.

### Edit the search line

The search line has a real caret. Press **Tab** or click the search line to
enter text mode; **Tab** or **Escape** leaves it. In text mode **←/→** move
the caret, **Home/End** jump to the ends, **Backspace** deletes before the
caret (**Ctrl+Backspace** deletes a word), and typing or pasting lands at the
caret. Outside text mode, keys navigate the list as before.

Dead keys work. On layouts like US international, `^`, `` ` ``, `~`, `'` and
`"` combine with the next key just as they do in any other app: `^` then `4` types `^4`
(so `2^4` is `16`), `^` then `a` types `â`, `'` then `c` types `ç`, and an
accent then space types the accent alone.

### Calculator

| Type | Get |
| --- | --- |
| `2+3` | `5` |
| `sqrt(144)+2^8` | `268` |
| `(1920*1080)/2` | `1036800` |

Operators `+ - * / % ^`, parentheses, unary signs, the constants `pi` `tau` `e`,
and the functions `sqrt cbrt abs round floor ceil exp ln log log2 sin cos tan
asin acos atan`.

The expression is parsed by hand rather than handed to the JavaScript engine
that runs your shell — nothing typed into the search field is ever evaluated as
code. A bare number stays a search term.

<img width="620" height="270" alt="image" src="https://github.com/user-attachments/assets/6105d637-7315-492e-b1c2-05b2fa274d4a" />

### Currency conversion

| Type | Get |
| --- | --- |
| `123 eur to usd` | `143.31 USD` · `123 EUR at 1.1651 · 28 Aug` |
| `123USD to eur` | `105.57 EUR` |
| `$120 to eur` | `102.99 EUR` |
| `50 euros to dollars` | `58.26 USD` |
| `(20+5) eur to usd` | `29.13 USD` |
| `eur to usd` | Today's rate for one unit |

<img width="611" height="268" alt="image" src="https://github.com/user-attachments/assets/df5c3676-fd7e-4ed7-8e86-781193adb4ca" />

166 currencies, any pair. Codes, symbols (`$ € £ ¥ ₹ ₽ ₴ zł`) and names all
work, and the amount can be any expression the calculator understands.

Rates come from [exchangerate-api's free daily
endpoint](https://open.er-api.com/v6/latest/EUR) — no API key — and are cached
at `~/.cache/omarchy/menu-exchange-rates.json`. **The fetch happens the first
time you actually type a conversion**, so a menu never used as a converter
never touches the network; after that it refreshes about once a day. Offline,
the row says so rather than showing a stale number without warning.

Commas are not accepted in amounts: `1,000` means one thousand in half of
Europe and one-point-nought in the other half, and neither reading is worth
guessing at. Crypto is not supported — the rate source is fiat only.

## Install

```bash
omarchy plugin add https://github.com/elamhut/ultracharged.menu --enable
omarchy restart shell
