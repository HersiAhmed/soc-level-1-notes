# Splunk

## The three components

I think of Splunk like a library.

| Component | Job | Library analogy |
|---|---|---|
| **Forwarder** | Lightweight agent on each device; collects and sends logs | Delivery truck |
| **Indexer** | Parses, normalizes, and stores logs as searchable events | Librarian who labels and shelves |
| **Search Head** | Where you search with SPL and build charts and dashboards | The search computer |

Flow: **Forwarder → Indexer → Search Head.** When you search, the search head asks the indexer, and the indexer returns matching events as field-value pairs.

Compared with EDR: forwarder = agent, and the console is split into indexer + search head.

## Navigating the home screen

- **Splunk Bar** (top): Messages, Settings, Activity (running searches), Help, Find, and an app switcher
- **Apps Panel** (left): installed apps. **Search & Reporting** is the default and where most work happens
- **Explore Splunk**: Add Data, Splunk Apps, Docs
- **Home Dashboard**: empty by default; your own dashboards show under "Yours"

## Getting data in

I uploaded a VPN log file (newline-delimited JSON) using **Add Data → Upload**:

1. Select source (the file)
2. Source type (JSON was detected automatically)
3. Input settings: **index** (the storage bucket) and **host** (which machine it came from)
4. Review
5. Done

Then in Search & Reporting, set the time picker to **All time**. The logs are old, so a recent time range shows nothing.

## SPL basics

The pipe `|` means "take these results and pass them to the next command." Most searches follow the same pattern:

> **get the data → parse (if needed) → filter → summarize**

**Count all events in an index**

```spl
index=my_vpn_index
| stats count
```

**Count events from one IP**

```spl
index=my_vpn_index
| search Source_ip="203.0.113.10"
| stats count
```

**Which users connected from an IP, and how often**

```spl
index=my_vpn_index
| search Source_ip="203.0.113.10"
| stats values(UserName) as UserName count
```

- `values(UserName)` lists the unique usernames
- `as UserName` labels the column
- `count` is the number of events

If more than one username shows up from the same IP, that could mean one attacker trying several stolen accounts.

**Exclude a value**

```spl
index=my_vpn_index Source_Country!="France"
| stats count
```

## What `stats` does

`stats` turns a pile of events into a summary table, like a pivot table.

| Command | Result |
|---|---|
| `stats count` | One total |
| `stats count by UserName` | A count per user |
| `stats values(UserName)` | Unique usernames |
| `stats dc(UserName)` | How many different users (distinct count) |

## What `spath` does

`spath` parses JSON into fields. The room's examples put it first as a safety net. I didn't need it because Splunk auto-detected the source type as JSON and the fields already showed up in the **Interesting Fields** sidebar.

Rule I use: if the fields are in the sidebar, skip `spath`. If they're missing, add it before filtering.

## Mistakes I made (and what they taught me)

**1. Searching the wrong index.** My upload landed in a different index than the one in the room's example searches, so those searches returned 0 results. The event data also had its own field called `index`, which Splunk renamed to `extracted_index`. That field is *data*, not the Splunk index the log is stored in.

Lesson: check the `index=` in the search bar first when results are empty.

**2. Putting a filter inside `stats`.**

```spl
index=my_vpn_index
| stats Source_Country="France" as France count     ← error
```

Splunk returned `The argument 'Source_Country=France' is invalid.` `stats` only accepts functions (`count`, `values()`, `dc()`, plus `by` and `as`). Filtering belongs in `search` or on the first line:

```spl
index=my_vpn_index
| search Source_Country="France"
| stats count
```

Lesson: **`search` picks the events, `stats` summarizes them.**

## Takeaways

- Time picker and index are the two most common reasons for "no results"
- Click a field in the sidebar to see its top values, a fast sanity check
- Order matters: parse → filter → summarize
