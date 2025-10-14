# BadBuddha NewsMarkers PRO - Instrument Filtering Guide

## Table of Contents
- [Overview](#overview)
- [Filter Modes](#filter-modes)
- [How Filtering Works](#how-filtering-works)
- [Auto By Instrument Mode](#auto-by-instrument-mode)
- [Custom Currency List Mode](#custom-currency-list-mode)
- [Currency Resolution System](#currency-resolution-system)
- [Special Cases](#special-cases)
- [Configuration Examples](#configuration-examples)
- [Debugging Filters](#debugging-filters)

---

## Overview

The **NewsMarkers PRO** indicator includes a sophisticated filtering system that allows you to show only relevant news events based on your trading instrument. This helps reduce chart clutter and focus on news that actually impacts your market.

### Key Features
- **Three filter modes**: None, Auto by Instrument, Custom Currency List
- **Automatic currency detection** from instrument names
- **Smart currency resolution** from country names
- **Special handling** for metals and unknown currencies
- **Debug mode** to verify filter behavior

---

## Filter Modes

The indicator supports three filtering modes via the `EventFilter` property:

### 1. **None** (`BBNewsFilterMode.None`)
- **No filtering applied**
- Shows **all High and Medium impact events**
- Best for multi-market traders or market overview

### 2. **Auto By Instrument** (`BBNewsFilterMode.AutoByInstrument`) ⭐ *DEFAULT*
- **Automatically detects currencies** relevant to your instrument
- Uses pattern matching on instrument name
- Ideal for focused trading on specific markets

### 3. **Custom Currency List** (`BBNewsFilterMode.CustomCurrencyList`)
- **Manual currency specification**
- You provide comma-separated currency codes
- Maximum control over which events display

---

## How Filtering Works

### Filter Pipeline

```
1. Load all events from CSV
     ↓
2. Filter by date range (PastDays, FutureDays)
     ↓
3. Filter by impact (High, Medium only)
     ↓
4. Apply currency filter (if enabled)
     ↓
5. Display filtered events
```

### Base Filtering (Always Applied)

**Before** any currency filtering, the indicator:
- Only includes events from `[Today - PastDays]` to `[Today + FutureDays]`
- Only includes **High** and **Medium** impact events
- Sorts events by LocalTime

### Currency Filtering

**Location**: `FilterEventsForRange()` method (line 1348)

```csharp
private List<NewsEvent> FilterEventsForRange(DateTime startDateInclusive, DateTime endDateInclusive)
{
    // 1. Get baseline (date range + High/Medium only)
    var baseline = allEvents
        .Where(e =>
            e.LocalTime.Date >= startDateInclusive &&
            e.LocalTime.Date <= endDateInclusive &&
            (e.Impact == "High" || e.Impact == "Medium"))
        .OrderBy(e => e.LocalTime);

    // 2. Apply currency filter based on mode
    if (EventFilter == BBNewsFilterMode.None)
        return baseline.ToList();

    // 3. Build allowed currency set
    HashSet<string> allowCurrencies = null;

    if (EventFilter == BBNewsFilterMode.CustomCurrencyList)
        allowCurrencies = ParseCustomCurrencies();
    else if (EventFilter == BBNewsFilterMode.AutoByInstrument)
        allowCurrencies = AutoCurrencySetForInstrument(Instrument.Name);

    // 4. Filter events by currency
    foreach (var e in baseline)
    {
        var resolved = ResolveEventCurrency(e.Currency, e.Country);
        if (allowCurrencies.Contains(resolved))
            filtered.Add(e);
    }

    return filtered;
}
```

---

## Auto By Instrument Mode

### How It Works

The `AutoCurrencySetForInstrument()` method analyzes your instrument name and automatically determines relevant currencies.

**Location**: Line 1438

### Supported Instruments

#### **CME FX Futures** (and Micros)
| Pattern | Currency | Examples |
|---------|----------|----------|
| `6A` or `M6A` | AUD | 6A, M6A, 6AH25 |
| `6B` or `M6B` | GBP | 6B, M6B, 6BM25 |
| `6C` or `M6C` | CAD | 6C, M6C, 6CZ25 |
| `6E` or `M6E` | EUR | 6E, M6E, 6EH25 |
| `6J` or `M6J` | JPY | 6J, M6J, 6JM25 |
| `6N` or `M6N` | NZD | 6N, M6N, 6NU25 |

#### **US Index Futures**
| Pattern | Currency | Examples |
|---------|----------|----------|
| `ES`, `MES` | USD | ES, MES, ES 03-25 |
| `NQ`, `MNQ` | USD | NQ, MNQ, NQ 06-25 |
| `YM`, `MYM` | USD | YM, MYM, YM 12-25 |
| `RTY`, `M2K` | USD | RTY, M2K, RTY 09-25 |

#### **Energy Futures**
| Pattern | Currency | Examples |
|---------|----------|----------|
| `CL`, `MCL` | USD | CL, MCL, CL 04-25 |
| `NG` | USD | NG, NG 01-25 |

#### **Treasury Futures**
| Pattern | Currency | Examples |
|---------|----------|----------|
| `ZB`, `ZN`, `ZT`, `ZF` | USD | ZB, ZN, ZT 03-25 |

#### **Metals Futures** (Special Handling)
| Pattern | Currency | Examples | Notes |
|---------|----------|----------|-------|
| `GC`, `MGC` | USD | GC, MGC, GC 02-25 | Gold |
| `SI` | USD | SI, SI 03-25 | Silver |
| `HG` | USD | HG, HG 05-25 | Copper |
| `PA`, `PL` | USD | PA, PL | Palladium, Platinum |

**Metals Note**: If `IncludeGlobalForMetals = true`, metals also include EUR and CNY news.

#### **Default Fallback**
- If instrument doesn't match any pattern → **USD** is used as default

### Code Example

```csharp
private HashSet<string> AutoCurrencySetForInstrument(string name)
{
    var s = (name ?? "").ToUpperInvariant();
    var set = new HashSet<string>(StringComparer.OrdinalIgnoreCase);

    // CME FX futures (and micros)
    if (s.Contains("6A") || s.Contains("M6A")) set.Add("AUD");
    if (s.Contains("6B") || s.Contains("M6B")) set.Add("GBP");
    if (s.Contains("6C") || s.Contains("M6C")) set.Add("CAD");
    if (s.Contains("6E") || s.Contains("M6E")) set.Add("EUR");
    if (s.Contains("6J") || s.Contains("M6J")) set.Add("JPY");
    if (s.Contains("6N") || s.Contains("M6N")) set.Add("NZD");

    // US index futures
    if (s.Contains("ES") || s.Contains("MES") ||
        s.Contains("NQ") || s.Contains("MNQ") ||
        s.Contains("YM") || s.Contains("MYM") ||
        s.Contains("RTY") || s.Contains("M2K"))
        set.Add("USD");

    // Treasuries, crude, natgas
    if (s.Contains("CL") || s.Contains("MCL") ||
        s.Contains("NG") ||
        s.Contains("ZB") || s.Contains("ZN") ||
        s.Contains("ZT") || s.Contains("ZF"))
        set.Add("USD");

    // Metals get USD
    if (IsMetalInstrument())
        set.Add("USD");

    // Default fallback
    if (set.Count == 0)
        set.Add("USD");

    return set;
}
```

### Example: Trading ES Futures

```
Instrument: ES 03-25
Auto-detected currencies: USD
Events shown: All High/Medium USD events
```

**Events You'll See:**
- ✅ US Fed Rate Decision (USD)
- ✅ US Non-Farm Payrolls (USD)
- ✅ US CPI (USD)

**Events You Won't See:**
- ❌ ECB Rate Decision (EUR)
- ❌ BOE Meeting (GBP)
- ❌ RBA Minutes (AUD)

---

## Custom Currency List Mode

### Configuration

Set `EventFilter = BBNewsFilterMode.CustomCurrencyList` and provide currencies via `CustomCurrenciesCsv`.

### Syntax

**Property**: `CustomCurrenciesCsv` (string)

**Supported Delimiters**: `,` `;` ` ` (space) `\t` (tab) `\r` `\n`

**Example Values**:
```
"USD,EUR,GBP"           // Standard
"USD, EUR, GBP"         // With spaces
"USD;EUR;GBP"           // Semicolon
"USD EUR GBP"           // Space-separated
"USD
EUR
GBP"                    // Multi-line
```

### Supported Currencies

The indicator supports these currencies (case-insensitive):

| Currency | Country/Region | ISO2 Code |
|----------|----------------|-----------|
| **USD** | United States | US |
| **EUR** | Eurozone | EU |
| **GBP** | United Kingdom | GB |
| **AUD** | Australia | AU |
| **NZD** | New Zealand | NZ |
| **CAD** | Canada | CA |
| **CHF** | Switzerland | CH |
| **JPY** | Japan | JP |
| **CNY** | China | CN |
| **CNH** | China (Offshore) | CN |
| **SEK** | Sweden | SE |
| **NOK** | Norway | NO |
| **DKK** | Denmark | DK |
| **TRY** | Türkiye | TR |
| **ZAR** | South Africa | ZA |
| **MXN** | Mexico | MX |
| **HKD** | Hong Kong | HK |
| **SGD** | Singapore | SG |
| **INR** | India | IN |
| **BRL** | Brazil | BR |
| **KRW** | South Korea | KR |

### Example: Multi-Currency Trader

```csharp
EventFilter = BBNewsFilterMode.CustomCurrencyList;
CustomCurrenciesCsv = "USD,EUR,GBP,JPY";
```

**Result**: Shows all High/Medium events for USD, EUR, GBP, and JPY.

---

## Currency Resolution System

The indicator uses a sophisticated system to match news events to currencies, since CSV files may have inconsistent currency/country data.

### Resolution Order

**Location**: `ResolveEventCurrency()` method (line 1118)

```
1. Check if event has Currency field
   ↓ If present → use it directly
   ↓ If missing → continue to step 2

2. Try to resolve from Country field
   ↓ Look up country in CountryAliasToIso2 dictionary
   ↓ Convert ISO2 code to currency
   ↓ If found → return currency
   ↓ If not found → continue to step 3

3. Parse Country field tokens
   ↓ Split by space, comma, parentheses, etc.
   ↓ Try each token in CountryAliasToIso2
   ↓ Check if token is a 3-letter currency code
   ↓ If found → return currency
   ↓ If not found → continue to step 4

4. Special case string matching
   ↓ Check for "UNITED STATES" → USD
   ↓ Check for "UNITED KINGDOM" → GBP
   ↓ Check for "EURO" → EUR
   ↓ Check for "CHINA" → CNY
   ↓ etc.
   ↓ If found → return currency
   ↓ If not found → return null
```

### Code Implementation

```csharp
private static string ResolveEventCurrency(string csvCurrency, string country)
{
    // Step 1: Direct currency field
    if (!string.IsNullOrWhiteSpace(csvCurrency))
        return csvCurrency.Trim().ToUpperInvariant();

    // Step 2: Country alias lookup
    var ctry = (country ?? "").Trim();
    if (!string.IsNullOrEmpty(ctry))
    {
        if (CountryAliasToIso2.TryGetValue(ctry, out var iso2) && !string.IsNullOrEmpty(iso2))
            if (Iso2ToCcy.TryGetValue(iso2, out var ccyFromIso))
                return ccyFromIso;

        // Step 3: Token parsing
        foreach (var tok in ctry.Split(new[] { ' ', ',', '(', ')', '/', '-', '.' }, StringSplitOptions.RemoveEmptyEntries))
        {
            if (CountryAliasToIso2.TryGetValue(tok, out var isoTok) && !string.IsNullOrEmpty(isoTok))
                if (Iso2ToCcy.TryGetValue(isoTok, out var ccyTok))
                    return ccyTok;

            // Check if token is a 3-letter currency code
            if (tok.Length == 3 && CcyToIso2.ContainsKey(tok.ToUpperInvariant()))
                return tok.ToUpperInvariant();
        }

        // Step 4: Special case matching
        string u = ctry.ToUpperInvariant();
        if (u.Contains("UNITED STATES")) return "USD";
        if (u.Contains("UNITED KINGDOM") || u.Contains("GREAT BRITAIN") || u.Contains("ENGLAND")) return "GBP";
        if (u.Contains("EURO")) return "EUR";
        if (u.Contains("CHINA")) return "CNY";
        if (u.Contains("HONG KONG")) return "HKD";
        if (u.Contains("KOREA")) return "KRW";
        if (u.Contains("TURKIYE") || u.Contains("TURKEY")) return "TRY";
    }

    return null; // Unable to resolve
}
```

### Country → Currency Mapping

The indicator maintains comprehensive country alias mappings:

**Location**: `BuildCountryAliasMap()` method (line 1036)

#### Example Mappings

```csharp
"United States"  → "US" → USD
"USA"            → "US" → USD
"U.S."           → "US" → USD

"United Kingdom" → "GB" → GBP
"UK"             → "GB" → GBP
"Great Britain"  → "GB" → GBP
"England"        → "GB" → GBP

"Eurozone"       → "EU" → EUR
"Euro Area"      → "EU" → EUR
"European Union" → "EU" → EUR

"China"          → "CN" → CNY
"Hong Kong"      → "HK" → HKD
"South Korea"    → "KR" → KRW
```

### Resolution Examples

| CSV Currency | CSV Country | Resolved | Logic Used |
|--------------|-------------|----------|------------|
| `USD` | (any) | `USD` | Direct field |
| ` ` | `United States` | `USD` | Country lookup |
| ` ` | `U.S.` | `USD` | Country alias |
| ` ` | `US CPI` | `USD` | Token parsing ("US") |
| ` ` | `Core Retail Sales (USA)` | `USD` | Token parsing ("USA") |
| ` ` | `UK GDP` | `GBP` | Token parsing ("UK") |
| ` ` | `European Union PMI` | `EUR` | String matching |
| ` ` | `Unknown Country` | `null` | No match |

---

## Special Cases

### 1. **Metals Special Handling**

**Property**: `IncludeGlobalForMetals` (bool, default = `false`)

When enabled and trading metals (GC, SI, HG, PA, PL):
- Adds **USD**, **EUR**, **CNY** to allowed currencies
- Recognizes that metals are global commodities affected by major economies

**Example**:
```csharp
// Trading Gold
Instrument: GC 02-25
EventFilter: AutoByInstrument
IncludeGlobalForMetals: true

Allowed currencies: USD, EUR, CNY

// You'll see:
✅ US CPI (USD)
✅ US Fed Rate Decision (USD)
✅ ECB Rate Decision (EUR)
✅ Eurozone CPI (EUR)
✅ China PMI (CNY)
```

**Code**:
```csharp
if (IncludeGlobalForMetals && IsMetalInstrument())
{
    allowCurrencies.Add("USD");
    allowCurrencies.Add("EUR");
    allowCurrencies.Add("CNY");
}

private bool IsMetalInstrument()
{
    var n = (Instrument?.MasterInstrument?.Name ?? Instrument?.FullName ?? "").ToUpperInvariant();
    return n.Contains("GC") || n.Contains("MGC") ||
           n.Contains("SI") || n.Contains("HG") ||
           n.Contains("PA") || n.Contains("PL");
}
```

### 2. **Unknown Currency Handling**

**Property**: `TreatUnknownCurrencyAsGlobal` (bool, default = `false`)

Events where currency cannot be resolved (returns `null` from `ResolveEventCurrency`):
- If `true` → **Include** the event (treat as globally relevant)
- If `false` → **Exclude** the event (conservative approach)

**Example**:
```
Event: "Global Risk Sentiment Survey" (Currency: blank, Country: "Global")
ResolveEventCurrency() returns: null

TreatUnknownCurrencyAsGlobal = true  → Event SHOWN
TreatUnknownCurrencyAsGlobal = false → Event HIDDEN
```

**Code**:
```csharp
foreach (var e in baseList)
{
    var resolved = ResolveEventCurrency(e.Currency, e.Country);

    if (string.IsNullOrWhiteSpace(resolved))
    {
        // Can't determine currency
        if (TreatUnknownCurrencyAsGlobal)
            filtered.Add(e);  // Include
        else
            continue;          // Exclude
    }
    else if (allowCurrencies.Contains(resolved))
    {
        filtered.Add(e);
    }
}
```

**Recommendation**:
- `false` (default) for clean, focused charts
- `true` if your CSV has many untagged global events

---

## Configuration Examples

### Example 1: ES Day Trader (Pure USD Focus)

```csharp
EventFilter = BBNewsFilterMode.AutoByInstrument;
IncludeGlobalForMetals = false;
TreatUnknownCurrencyAsGlobal = false;

// Result: Only USD High/Medium events
```

### Example 2: FX Trader (EUR/USD)

```csharp
EventFilter = BBNewsFilterMode.CustomCurrencyList;
CustomCurrenciesCsv = "USD,EUR";
TreatUnknownCurrencyAsGlobal = false;

// Result: Only USD and EUR High/Medium events
```

### Example 3: Multi-Market Trader (Major Pairs)

```csharp
EventFilter = BBNewsFilterMode.CustomCurrencyList;
CustomCurrenciesCsv = "USD,EUR,GBP,JPY,AUD,CAD";
TreatUnknownCurrencyAsGlobal = true; // Include global events

// Result: All major currency events + global events
```

### Example 4: Gold Trader (Global Macro Focus)

```csharp
EventFilter = BBNewsFilterMode.AutoByInstrument;
IncludeGlobalForMetals = true;
TreatUnknownCurrencyAsGlobal = true;

// Result: USD + EUR + CNY events + unknown/global events
```

### Example 5: Show Everything

```csharp
EventFilter = BBNewsFilterMode.None;

// Result: All High/Medium events regardless of currency
```

---

## Debugging Filters

### Enable Debug Mode

**Property**: `EnableDebug` (bool)

When enabled, the indicator prints detailed filtering information to the NinjaTrader Output window.

### Debug Output Example

```
[NewsPRO][08:30:15.123] Filter base: range=[2025-10-05..2025-10-15] count=87
[NewsPRO][08:30:15.125] Filter mode: AutoByInstrument 'ES 03-25' → allow=[USD]
[NewsPRO][08:30:15.130] Filter result: kept=34 removed=53 unknownKept=0 unknownDropped=3
    sampleKept=[10-09 08:30-USD-High-Non-Farm Payrol… | 10-09 10:00-USD-High-ISM Services PM… | ...]
    sampleRemoved=[10-09 09:00-EUR-High-ECB Rate Deci… | 10-10 03:00-AUD-Medium-RBA Minutes | ...]
```

### Debug Output Fields

| Field | Description |
|-------|-------------|
| `range` | Date range being filtered |
| `count` | Total events in date range (before currency filter) |
| `Filter mode` | Active filter mode |
| `allow` | Allowed currencies for this filter |
| `kept` | Number of events passing filter |
| `removed` | Number of events filtered out |
| `unknownKept` | Unknown currency events included |
| `unknownDropped` | Unknown currency events excluded |
| `sampleKept` | First 5 kept events |
| `sampleRemoved` | First 5 removed events |

### Using Debug Mode

1. Set `EnableDebug = true` in indicator properties
2. Reload the indicator or change instrument
3. Open **Tools → Output Window** (Ctrl+F2)
4. Look for `[NewsPRO]` messages
5. Verify the `allow` currencies match your expectations
6. Check `kept` vs `removed` counts
7. Review sample events to confirm filtering logic

### Verification Checklist

- [ ] Correct currencies detected for my instrument?
- [ ] Expected events in `sampleKept`?
- [ ] Unwanted events in `sampleRemoved`?
- [ ] Unknown events handled correctly?
- [ ] Metals special handling working (if applicable)?

---

## Property Reference

### Filter-Related Properties

| Property | Type | Group | Default | Description |
|----------|------|-------|---------|-------------|
| `EventFilter` | `BBNewsFilterMode` | 06. Filtering | `AutoByInstrument` | Filter mode |
| `CustomCurrenciesCsv` | `string` | 06. Filtering | `""` | Custom currencies (comma-separated) |
| `IncludeGlobalForMetals` | `bool` | 06. Filtering | `false` | Add USD/EUR/CNY for metals |
| `TreatUnknownCurrencyAsGlobal` | `bool` | 06. Filtering | `false` | Include events with unknown currency |
| `EnableDebug` | `bool` | 99. Advanced | `false` | Print debug output |

---

## Summary

### When to Use Each Mode

| Mode | Best For | Pros | Cons |
|------|----------|------|------|
| **None** | Multi-market monitoring | See everything | Chart clutter |
| **Auto By Instrument** | Single instrument focus | Zero configuration | Limited instrument support |
| **Custom Currency List** | Specific currency sets | Full control | Manual configuration |

### Quick Tips

1. **Start with Auto mode** for most instruments
2. **Enable debug** to understand what's being filtered
3. **Use Custom mode** for FX pairs or multi-market strategies
4. **Enable metals special handling** when trading GC/SI/HG
5. **Keep TreatUnknownCurrencyAsGlobal OFF** unless your CSV has many untagged events
6. **Test your filter** with EnableDebug after changing instruments

---

## Troubleshooting

### Problem: No events showing

**Check**:
1. Is `ShowVerticalLines` or `ShowEventsTable` enabled?
2. Are there High/Medium events in your date range?
3. Is `EventFilter` set to `None` temporarily to see all events?
4. Enable `EnableDebug` and check allowed currencies

### Problem: Wrong events showing

**Check**:
1. Verify `EventFilter` mode
2. Check `CustomCurrenciesCsv` for typos
3. Enable `EnableDebug` and verify `allow` currencies
4. Check if `TreatUnknownCurrencyAsGlobal` is affecting results

### Problem: Instrument not recognized

**Solution**:
- Auto mode doesn't recognize your instrument
- Use **Custom Currency List** mode instead
- Manually specify relevant currencies

### Problem: Missing some expected events

**Check**:
1. Are they High or Medium impact? (Low events are always filtered)
2. Is the event within your date range (`PastDays`, `FutureDays`)?
3. Is the currency correctly resolved? (Enable debug to check)
4. Is `TreatUnknownCurrencyAsGlobal` needed?

---

## Code Reference

### Key Methods

| Method | Line | Purpose |
|--------|------|---------|
| `FilterEventsForRange()` | 1348 | Main filtering logic |
| `AutoCurrencySetForInstrument()` | 1438 | Auto-detect currencies from instrument |
| `ResolveEventCurrency()` | 1118 | Resolve event currency from CSV data |
| `IsMetalInstrument()` | 1430 | Detect metal futures |
| `BuildCountryAliasMap()` | 1036 | Build country → currency mappings |

### Key Data Structures

| Dictionary | Line | Purpose |
|------------|------|---------|
| `CcyToIso2` | 131 | Currency → ISO2 code (for flags) |
| `Iso2ToCcy` | 142 | ISO2 code → Currency (inverse) |
| `CountryAliasToIso2` | 139 | Country name/alias → ISO2 code |

---

**Document Version**: 1.0
**Compatible With**: BadBuddha_NewsMarkers_PRO v3.8.0.0+
**Last Updated**: October 2025
