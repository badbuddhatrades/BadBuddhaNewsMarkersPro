# BadBuddha NewsMarkers PRO

<div align="center">

![Version](https://img.shields.io/badge/version-v4.1.0-blue)
![Platform](https://img.shields.io/badge/platform-NinjaTrader%208-green)
![License](https://img.shields.io/badge/license-NinjaTrader%20Vendor-orange)
![Product ID](https://img.shields.io/badge/Product%20ID-331-purple)

**Professional economic news calendar with visual markers, blackout windows, and live countdowns**

[Features](#-key-features) • [Installation](#-installation) • [Settings Guide](#-settings-guide) • [Use Cases](#-use-cases) • [Free Trial](#-14-day-free-trial)

</div>

---

## Table of Contents

- [Overview](#-overview)
- [Why Traders Use This](#-why-traders-use-this)
- [Key Features](#-key-features)
- [PRO vs Free Comparison](#-pro-vs-free-comparison)
- [Installation](#-installation)
- [Settings Guide](#-settings-guide)
- [Use Cases](#-use-cases)
- [Troubleshooting](#-troubleshooting)
- [14-Day Free Trial](#-14-day-free-trial)
- [Support](#-support)

---

## Overview

**BadBuddha NewsMarkers PRO** is a professional-grade news calendar indicator for NinjaTrader 8 that helps traders avoid getting caught in high-volatility news events. The indicator automatically marks economic news releases directly on your charts with visual markers, blackout windows, and live countdowns - so you always know when to stay out of the market.

### Version Information

| Info | Details |
|------|---------|
| **Version** | v4.2.0 |
| **Edition** | PRO (Licensed) |
| **Author** | BadBuddha Customs LLC |


---

## Why Traders Use This

**Stop getting stopped out by news events you didn't see coming.**

- **Avoid News Whipsaws** - Visual warnings before high-impact news hits
- **Protect Your Capital** - Automatic blackout windows show when NOT to trade
- **Stay Informed** - Live countdown timers to the next major event
- **Focus on Trading** - Automated updates, no manual calendar checking
- **Multi-Timeframe** - Works on any chart from tick to daily
- **Strategy-Ready** - Programmatic access for automated trading systems

## Key Features

### 1. Visual News Markers

Mark high-impact economic releases directly on your charts:

- Vertical lines at exact event times
- High and Medium impact filtering
- Customizable colors and opacity
- Optional event title labels with countdown timers
- Country flags for currency identification

### 2. Blackout Windows

Visual "no-trade zones" around news events:

- Shaded rectangles showing danger periods
- Separate pre/post windows for High and Medium impact
- Configurable minutes before and after (default: 5 min)
- Adjustable transparency to keep charts readable

### 3. Live Events Table

Real-time table showing upcoming news:

- "Due In" countdown column
- Actual/Forecast/Previous data values
- Flexible positioning (4 corners)
- Overlay on chart or separate popout window
- Customizable row count and sizing

### 4. Currency Filtering (NEW in v4.1)

Focus only on currencies you trade:

- Checkbox selection for 9 major currencies
- USD, EUR, GBP, AUD, NZD, CAD, CHF, JPY, CNY
- No typing required - simple on/off toggles
- Perfect for forex and currency futures traders
- Enable/disable entire filter with one switch

### 5. Real-Time Alerts

Never miss an important news event:

- Audio alerts at multiple intervals (e.g., 15, 5, 1 min)
- Separate settings for High vs Medium impact
- Customizable alert messages with placeholders
- Optional trading hours restriction
- Duplicate alert prevention

### 6. Strategy Integration

Programmatic access for automated systems:

```csharp
// Check if currently in news blackout
if (newsIndicator.IsInNewsWindow) return;

// Minutes to next high impact event
int minutes = newsIndicator.MinutesToNextHighImpact;

// Get upcoming event title
string title = newsIndicator.UpcomingEventTitle;
```

Perfect for protecting automated strategies from news volatility.

### 7. Auto-Updates

Set it and forget it:

- Configurable refresh intervals (default: 5 min)
- Built-in version checker for indicator updates
- File system monitoring for instant reloads
- No manual calendar management

---

## PRO vs Free Comparison

| Feature | Free Version | PRO Version |
|---------|--------------|-------------|
| **Visual Markers** | High impact only | High + Medium impact |
| **Blackout Windows** | Basic | Advanced with separate High/Medium settings |
| **Events Table** | Basic overlay | Advanced with popout, A/F/P data |
| **Currency Filtering** | None | 9 currencies with checkbox control |
| **Live Countdowns** | No | Yes - real-time updates |
| **Alerts** | No | Yes - multi-interval audio alerts |
| **Strategy Integration** | No | Full API access |
| **Auto CSV Updates** | Manual only | Automatic from cloud |
| **Date Range Control** | Fixed | Customizable past/future days |
| **Update Checker** | 7-day check | 1-day check with popup |
| **Support** | Community | Priority support |

---

## Installation

### Prerequisites

- NinjaTrader 8.1 or higher
- Windows operating system
- Internet connection for calendar updates

### Installation Steps

1. **Download**
   - Download `BadBuddha_NewsMarkersPRO.zip` from BadBuddha Customs

2. **Import to NinjaTrader**
   ```
   1. Open NinjaTrader Control Center
   2. Tools > Import > NinjaScript Add-On
   3. Browse to the .zip file
   4. Click Import
   5. Wait for "Import successful"
   ```

3. **Add to Chart**
   ```
   1. Right-click any chart
   2. Select Indicators
   3. Find "BadBuddha_NewsMarkers_PRO"
   4. Click Add
   5. Configure settings
   ```

4. **First-Time Setup**
   - Start with default settings
   - Enable currency filter if trading forex
   - Adjust blackout windows to your risk tolerance
   - Configure alerts if desired

---

## Settings Guide

### Data Source

| Setting | Description | Default |
|---------|-------------|---------|
| **Trading Mode** | Realtime (current week) or Historical (full calendar) | Realtime |
| **Auto Reload Minutes** | How often to check for updates | 5 |
| **Force CSV Download** | Force immediate re-download | false |

### Display Options

| Setting | Description | Default |
|---------|-------------|---------|
| **Show Vertical Lines** | Display lines at event times | true |
| **Line Opacity (%)** | Line transparency | 25 |
| **Line Thickness (px)** | Line width | 3 |
| **Show Event Labels** | Display event titles | false |
| **Show Countdown Labels** | Live countdown timers | false |

### Blackout Windows

| Setting | Description | Default |
|---------|-------------|---------|
| **Enable Blackout High** | Show High impact blackouts | true |
| **High: Pre/Post (min)** | Minutes before/after High events | 5 / 5 |
| **Enable Blackout Medium** | Show Medium impact blackouts | true |
| **Medium: Pre/Post (min)** | Minutes before/after Medium events | 3 / 3 |
| **Blackout Opacity (%)** | Rectangle transparency | 15 |

### Currency Filter (NEW)

| Setting | Description | Default |
|---------|-------------|---------|
| **Enable Currency Filter** | Master toggle | false |
| **Show [Currency]** | Individual checkboxes for USD, EUR, GBP, AUD, NZD, CAD, CHF, JPY, CNY | All true |

**Example:** To see only USD and EUR events, enable the filter and check only USD and EUR boxes.

### Events Table

| Setting | Description | Default |
|---------|-------------|---------|
| **Show Events Table** | Enable table display | true |
| **Table Display Mode** | Overlay, Popout, or Both | Overlay |
| **Max Rows** | Number of events shown | 12 |
| **Corner** | TopLeft, TopRight, BottomLeft, BottomRight | TopLeft |

### Alerts

| Setting | Description | Default |
|---------|-------------|---------|
| **Enable Alerts** | Turn on audio alerts | false |
| **Alert High/Medium** | Which impacts trigger alerts | High: true, Medium: false |
| **Alert Windows** | Alert times before event (CSV) | "15,5" |
| **Alert Message Format** | Customize with placeholders | "{IMPACT} Impact in {MINUTES} min: {TITLE}" |

---

## Use Cases

### Day Trading Index Futures (ES, NQ, YM)

**Setup:**
- Enable USD filter only
- Blackout: High Pre/Post = 5 min
- Alerts: 15 min, 5 min warnings
- Events table in TopRight corner

**Why:** Avoid Fed announcements, CPI, NFP, and other USD events that cause massive spikes.

### Forex Trading (EUR/USD, GBP/USD)

**Setup:**
- Enable USD, EUR, GBP filters
- Blackout: High = 5 min, Medium = 3 min
- Show countdown labels on chart
- Alerts during trading hours only

**Why:** Currency pairs react to news from both base and quote currencies.

### Automated Strategy Protection

```csharp
// In your strategy OnBarUpdate():
BadBuddha_NewsMarkersPRO news = Indicators.BadBuddha_NewsMarkersPRO();

// Don't enter new positions during blackout
if (news.IsInNewsWindow) return;

// Close positions 15 min before high impact news
if (Position.MarketPosition != MarketPosition.Flat
    && news.MinutesToNextHighImpact < 15)
{
    ExitLong();
    ExitShort();
}
```

### Overnight Trading

**Setup:**
- Enable all relevant currencies
- Date Range: Past Days = 0, Future Days = 1
- Trading Mode: Realtime

**Why:** See tomorrow's news events to avoid holding positions through major releases.

---

## Troubleshooting

### Events Not Showing

**Check:**
- CSV file exists in `Documents\NinjaTrader 8\` folder
- Trading Mode matches your needs (Realtime vs Historical)
- Date Range includes today (Past Days ≥ 0)
- Currency filter not blocking all events

**Solution:**
- Enable "Force CSV Download" and reload chart
- Check Output window (Tools > Output Window) for errors

### Wrong Event Times

**Check:**
- Computer timezone settings
- Chart time vs wall clock time
- Countdown Clock setting (Auto, WallClock, ChartTime)

**Solution:**
- Set Countdown Clock to "WallClock" for local time
- Verify CSV timezone is UTC (default)

### Blackout Windows Not Appearing

**Check:**
- "Enable Blackout High/Medium" is checked
- Events are High or Medium impact
- Blackout Opacity > 0

**Solution:**
- Increase Blackout Opacity to 25% for visibility
- Verify events exist with Indicators > Properties > Debug Mode

### Currency Filter Not Working

**Check:**
- "Enable Currency Filter" is checked
- At least one currency checkbox is enabled
- Events have currency data in CSV

**Solution:**
- Disable filter temporarily to see all events
- Re-enable and check only desired currencies

---

## 14-Day Free Trial

### Try Before You Buy

**BadBuddha NewsMarkers PRO** includes a **14-day free trial** with complete functionality:

- All features unlocked
- No credit card required
- No feature limitations
- Full support during trial

After 14 days, a license is required to continue using the indicator.

### How to Activate Trial

1. Install the indicator following the installation steps above
2. Add to any chart
3. Trial automatically begins on first use
4. Monitor trial days remaining in indicator properties

### Purchase After Trial

Contact BadBuddha Customs to purchase a full license and continue using all PRO features.

---

## Support

### Getting Help

- **Website**: [BadBuddha Customs](https://badbuddhacustoms.com)
- **Email Support**: support@badbuddhacustoms.com
- **Update Notifications**: Automatic via indicator (checks daily)

### Staying Updated

The indicator includes automatic update checking:
- Checks for new versions every 24 hours
- Displays popup with version info and changelog
- Direct download link to latest release
- Can be disabled in settings (Group 10)

---

## Summary

**BadBuddha NewsMarkers PRO** is the complete news calendar solution for traders who need:

- Visual warnings for high-volatility events
- Customizable blackout windows
- Real-time countdown timers
- Currency filtering for focused trading
- Strategy API for automated systems
- Automated data updates
- Priority support

**Perfect for:**
- Day traders (ES, NQ, YM, CL, GC)
- Forex traders (all major pairs)
- Automated strategy developers
- Multi-timeframe traders
- Anyone who wants to avoid news whipsaws

---

<div align="center">

### Try Risk-Free for 14 Days

**Stop losing money to unexpected news events. Protect your capital today.**

[Contact BadBuddha Customs](https://badbuddhacustoms.com)

---

**Made with precision by BadBuddha Customs LLC**

*Professional NinjaTrader 8 Tools for Serious Traders*

Copyright © 2025 BadBuddha Customs LLC

---

**Disclaimer**: This indicator is provided for informational purposes only. Trading involves substantial risk of loss. Past performance is not indicative of future results. Always conduct your own research and risk assessment before trading.

</div>
