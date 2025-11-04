# BadBuddha NewsMarkers PRO

## Overview

BadBuddha NewsMarkers PRO is a professional-grade news calendar indicator for NinjaTrader 8 that helps traders avoid getting caught in high-volatility news events. The indicator automatically marks economic news releases directly on your charts with visual markers, blackout windows, and live countdowns.

## Why Traders Use This

- **Avoid News Whipsaws**: Know exactly when high-impact news is about to hit before your positions get stopped out
- **Protect Your Capital**: Automatically visualize blackout windows where trading should be avoided
- **Stay Informed**: Live countdown timers show exactly how many minutes until the next major news event
- **Automated Updates**: No manual calendar checking - news data automatically downloads and updates
- **Multi-Timeframe**: Works on any chart timeframe from tick charts to daily charts
- **Strategy Integration**: Access news data programmatically from your automated strategies

## Key Features

### Visual News Markers
- High and Medium impact news events displayed as vertical lines on your chart
- Customizable colors and opacity for each impact level
- Optional event labels showing the news title and countdown
- Country flags displayed for visual currency identification

### Blackout Windows
- Shaded rectangular zones showing pre/post event danger periods
- Separate settings for High and Medium impact events
- Configurable minutes before and after each event
- Adjustable opacity to keep your chart clean

### Events Table
- Live table overlay showing upcoming news events
- "Due In" column with countdown timers
- Display Actual/Forecast/Previous data values
- Flexible positioning (top-left, top-right, bottom-left, bottom-right)
- Can be shown as overlay on chart or separate popout window

### Currency Filtering
- Filter events by specific currencies (USD, EUR, GBP, AUD, NZD, CAD, CHF, JPY, CNY)
- Perfect for forex traders focusing on specific pairs
- Simple checkbox interface - no typing required
- Enable/disable filter with a single toggle

### Real-Time Alerts
- Audio alerts at configurable intervals before news events (e.g., 15 min, 5 min, 1 min)
- Separate alert settings for High and Medium impact
- Customizable alert messages with event details
- Option to only alert during trading hours

### Strategy Integration
- `IsInNewsWindow` property - check if currently in a blackout period
- `MinutesToNextHighImpact` - minutes until next high impact event
- `MinutesToNextMediumImpact` - minutes until next medium impact event
- `UpcomingEventTitle` - name of the next upcoming event
- Perfect for automated trading systems that need to avoid news

### Auto-Updates
- Automatic CSV download from Google Drive
- Configurable update intervals (default: every 5 minutes)
- Built-in version checker notifies you of indicator updates
- File system monitoring for instant reload on manual CSV updates

## Settings Guide

### 01. Data Source
- **Trading Mode**: Choose between Realtime (current week only) or Historical (full calendar)
- **CSV File Name**: Name of the calendar CSV file
- **Auto Reload Minutes**: How often to check for CSV updates
- **Google Drive URL**: Source URL for automatic calendar downloads
- **Force CSV Download**: Force immediate re-download of calendar data

### 02. Display
- **Show Vertical Lines**: Display vertical lines at news event times
- **Line Opacity (%)**: Transparency of vertical lines (0-100)
- **Line Thickness (px)**: Width of vertical lines in pixels
- **Show Event Labels**: Display event title and details near markers
- **Show Countdown Labels**: Show live countdown timers on labels
- **Label Title Max Chars**: Truncate long titles to this length
- **Countdown Clock**: Use wall clock time, chart time, or auto-detect

### 03. Blackout Windows
- **Enable Blackout High**: Show blackout zones for high impact events
- **Enable Blackout Medium**: Show blackout zones for medium impact events
- **High: Pre (min)**: Minutes before high impact event to start blackout
- **High: Post (min)**: Minutes after high impact event to end blackout
- **Medium: Pre (min)**: Minutes before medium impact event to start blackout
- **Medium: Post (min)**: Minutes after medium impact event to end blackout
- **Blackout Opacity (%)**: Transparency of blackout rectangles

### 04. Currency Filter
- **Enable Currency Filter**: Master toggle to enable currency filtering
- **Show USD Events**: Display events for US Dollar
- **Show EUR Events**: Display events for Euro
- **Show GBP Events**: Display events for British Pound
- **Show AUD Events**: Display events for Australian Dollar
- **Show NZD Events**: Display events for New Zealand Dollar
- **Show CAD Events**: Display events for Canadian Dollar
- **Show CHF Events**: Display events for Swiss Franc
- **Show JPY Events**: Display events for Japanese Yen
- **Show CNY Events**: Display events for Chinese Yuan

### 05. Date Range
- **Past Days**: How many days in the past to load events (0 = today only)
- **Future Days**: How many days in the future to load events (0 = today only)

### 08. Events Table
- **Show Events Table**: Enable the upcoming events table
- **Table Display Mode**: Choose Overlay (on chart), Popout (separate window), or Both
- **Max Rows**: Maximum number of events to show in table
- **Width (px)**: Table width in pixels
- **Row Height (px)**: Height of each row
- **Opacity (%)**: Table background transparency
- **Corner**: Position on chart (TopLeft, TopRight, BottomLeft, BottomRight)
- **Title Max Chars**: Maximum characters for event titles
- **Vertical Offset (px)**: Distance from edge of chart
- **Show Data Columns (A/F/P)**: Display Actual/Forecast/Previous values

### 09. Alerts
- **Enable Alerts**: Turn on audio alerts for news events
- **Alert High Impact Events**: Fire alerts for high impact news
- **Alert Medium Impact Events**: Fire alerts for medium impact news
- **Alert Windows (minutes)**: Comma-separated alert times (e.g., "15,5,1")
- **Alert Sound File**: Name of WAV file to play
- **Alert Priority**: Alert priority level
- **Alert Only During Trading Hours**: Restrict alerts to market hours
- **Alert Message Format**: Customize alert text with placeholders

### 10. Updates
- **Enable Update Check**: Check for new indicator versions
- **Show Popup**: Display popup window when updates are available

## How to Use

1. **Install**: Import the indicator into NinjaTrader 8
2. **Add to Chart**: Apply "BadBuddha_NewsMarkers_PRO" to any chart
3. **Configure**: Adjust settings in the indicator properties panel
4. **Currency Filter** (Optional): Enable currency filter and select which currencies you want to track
5. **Monitor**: Watch for vertical lines, blackout windows, and countdown timers
6. **Integrate** (Optional): Access news data from your strategies using the provided properties

## 14-Day Free Trial

Try BadBuddha NewsMarkers PRO **free for 14 days** with full functionality. No credit card required.

After the trial period, a license is required to continue using the indicator.

## Getting More Information

- **Support**: For technical support and questions, please contact BadBuddha Customs
- **Updates**: The indicator automatically checks for updates and will notify you when new versions are available


## System Requirements

- NinjaTrader 8.1 or higher
- Windows operating system
- Internet connection for automatic calendar updates

---

**Disclaimer**: This indicator is provided for informational purposes only. Trading involves substantial risk of loss. Past performance is not indicative of future results. Always conduct your own research and risk assessment before trading.
