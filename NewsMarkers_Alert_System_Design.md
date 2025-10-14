# BadBuddha NewsMarkers PRO - Alert System Design

## Overview

Design document for implementing a comprehensive alerting system for news events in the NewsMarkers PRO indicator.

**Version**: 1.0
**Date**: October 2025
**Status**: Design Phase

---

## Table of Contents

- [Requirements](#requirements)
- [Architecture](#architecture)
- [Alert Types](#alert-types)
- [Configuration](#configuration)
- [Implementation Details](#implementation-details)
- [Discord Integration (Future)](#discord-integration-future)
- [Testing Strategy](#testing-strategy)

---

## Requirements

### Phase 1: Audio Alerts (Immediate)

#### Must Have
- ✅ Audio alert X minutes before High impact events
- ✅ Audio alert X minutes before Medium impact events
- ✅ Configurable alert windows (e.g., 15min, 5min, 1min before)
- ✅ No duplicate alerts for same event
- ✅ Respect existing event filters (AutoByInstrument, CustomCurrencyList)
- ✅ Alert only during enabled hours (optional trading hours filter)
- ✅ Separate enable/disable for High vs Medium alerts
- ✅ Use NinjaTrader's built-in Alert() method

#### Nice to Have
- ⭐ Multiple alert windows per event (e.g., 15min + 5min + 1min)
- ⭐ Custom alert sound selection
- ⭐ Alert priority levels
- ⭐ Alert message includes event title, time, currency, impact
- ⭐ "Alert once per session" vs "Alert on every occurrence"
- ⭐ Visual indicator when alert fires (flash, color change)

### Phase 2: Discord Integration (Future)

#### Must Have
- 🔮 Send alert messages to Discord webhook
- 🔮 Configurable Discord webhook URL
- 🔮 Rich embed formatting (title, color, fields)
- 🔮 Include event details (time, currency, impact, title, forecast data)
- 🔮 Async HTTP POST to avoid blocking chart

#### Nice to Have
- 🔮 Discord bot with commands (@mention for next events)
- 🔮 Multiple Discord webhooks for different impact levels
- 🔮 Discord role mentions for High impact events
- 🔮 Include chart screenshot in Discord message
- 🔮 Rate limiting / throttling

---

## Architecture

### High-Level Design

```
┌─────────────────────────────────────────────────────────┐
│                  OnBarUpdate() Cycle                     │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
         ┌────────────────────────────┐
         │  Get Filtered Events        │
         │  (Existing Filter Logic)    │
         └────────────┬───────────────┘
                      │
                      ▼
         ┌────────────────────────────┐
         │   Check Alert Windows       │
         │   For Each Event:           │
         │   - Calculate minutes until │
         │   - Check if in alert range │
         │   - Check if already alerted│
         └────────────┬───────────────┘
                      │
                      ▼
         ┌────────────────────────────┐
         │    Fire Alerts              │
         │    - Audio Alert            │
         │    - Log to Output          │
         │    - (Future: Discord)      │
         │    - Track alerted events   │
         └─────────────────────────────┘
```

### Alert State Tracking

Need to track which events have been alerted on to prevent duplicates:

```csharp
// Alert tracking structure
private class AlertRecord
{
    public DateTime EventTime;      // Event LocalTime
    public string EventTitle;       // Event Title
    public string Impact;           // Event Impact
    public int AlertWindow;         // Alert window (minutes before)
    public DateTime AlertedAt;      // When we fired the alert
}

private List<AlertRecord> _alertHistory = new List<AlertRecord>();
```

### Alert Window System

Support multiple alert windows per event:

```
Event Time: 10:00:00
Alert Windows: [15, 5, 1]

Timeline:
09:45:00 ──► First Alert  (15 min before)
09:55:00 ──► Second Alert (5 min before)
09:59:00 ──► Final Alert  (1 min before)
10:00:00 ──► EVENT OCCURS
```

---

## Alert Types

### 1. Audio Alerts (Phase 1)

**NinjaTrader Built-in Alert Method**:
```csharp
Alert(string alertName, Priority priority, string message,
      string soundFile, int rearmSeconds, Brush backColor, Brush foreColor)
```

**Parameters**:
- `alertName`: Unique identifier (use event key)
- `priority`: `Priority.High` or `Priority.Medium` or `Priority.Low`
- `message`: Alert message text
- `soundFile`: Path to .wav file or built-in sound
- `rearmSeconds`: Seconds before same alert can fire again (use 0)
- `backColor/foreColor`: Alert window colors

**Built-in NinjaTrader Sounds**:
- `Alert1.wav` through `Alert4.wav` (default sounds)
- Custom .wav files from user folder

**Example**:
```csharp
Alert("NFP_15min",
      Priority.High,
      "High Impact: Non-Farm Payrolls in 15 minutes (USD 08:30)",
      @"Alert2.wav",
      0,
      Brushes.Red,
      Brushes.White);
```

### 2. Discord Webhooks (Phase 2)

**Webhook Structure**:
```json
{
  "embeds": [{
    "title": "📢 News Alert: High Impact Event",
    "description": "Non-Farm Payrolls (USD)",
    "color": 15158332,
    "fields": [
      { "name": "Time", "value": "08:30 EST", "inline": true },
      { "name": "Impact", "value": "High", "inline": true },
      { "name": "Currency", "value": "USD", "inline": true },
      { "name": "Alert Window", "value": "15 minutes", "inline": true },
      { "name": "Forecast", "value": "200K", "inline": true },
      { "name": "Previous", "value": "185K", "inline": true }
    ],
    "timestamp": "2025-10-09T12:15:00.000Z"
  }]
}
```

**HTTP POST Implementation**:
```csharp
private async Task SendDiscordAlert(NewsEvent ev, int minutesBefore)
{
    using (var client = new HttpClient())
    {
        var payload = new {
            embeds = new[] {
                new {
                    title = $"📢 {ev.Impact} Impact News Alert",
                    description = ev.Title,
                    color = GetDiscordColor(ev.Impact),
                    fields = new[] {
                        new { name = "Time", value = ev.LocalTime.ToString("HH:mm"), inline = true },
                        new { name = "Currency", value = ev.Currency, inline = true },
                        new { name = "Alert", value = $"{minutesBefore} min before", inline = true }
                    }
                }
            }
        };

        var json = JsonConvert.SerializeObject(payload);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        await client.PostAsync(DiscordWebhookUrl, content);
    }
}

private int GetDiscordColor(string impact)
{
    // Discord colors are decimal RGB values
    if (impact == "High") return 15158332;   // Red (0xE74C3C)
    if (impact == "Medium") return 15105570; // Orange (0xE67E22)
    return 3447003;                          // Blue (0x3498DB)
}
```

---

## Configuration

### Alert Properties (Phase 1)

```csharp
// ============================================
// Alert Configuration (Group: "09. Alerts")
// ============================================

[Display(Name="Enable Alerts", GroupName="09. Alerts", Order=0)]
public bool EnableAlerts { get; set; } = false;

[Display(Name="Alert High Impact Events", GroupName="09. Alerts", Order=1)]
public bool AlertHighImpact { get; set; } = true;

[Display(Name="Alert Medium Impact Events", GroupName="09. Alerts", Order=2)]
public bool AlertMediumImpact { get; set; } = false;

[Display(Name="Alert Windows (minutes)", GroupName="09. Alerts", Order=3,
    Description="Comma-separated alert times before event (e.g., 15,5,1)")]
public string AlertWindowsCsv { get; set; } = "15,5";

[Display(Name="Alert Sound File", GroupName="09. Alerts", Order=4,
    Description="Sound file name (e.g., Alert2.wav) or full path")]
public string AlertSoundFile { get; set; } = "Alert2.wav";

[Display(Name="Alert Priority", GroupName="09. Alerts", Order=5)]
public Priority AlertPriority { get; set; } = Priority.High;

[Display(Name="Respect Trading Hours", GroupName="09. Alerts", Order=6,
    Description="Only alert during instrument trading hours")]
public bool AlertOnlyDuringTradingHours { get; set; } = false;

[Display(Name="Alert Message Format", GroupName="09. Alerts", Order=7,
    Description="Use {IMPACT}, {TITLE}, {CURRENCY}, {TIME}, {MINUTES}")]
public string AlertMessageFormat { get; set; } =
    "{IMPACT} Impact in {MINUTES} min: {TITLE} ({CURRENCY} {TIME})";
```

### Discord Properties (Phase 2)

```csharp
// ============================================
// Discord Integration (Group: "10. Discord")
// ============================================

[Display(Name="Enable Discord Alerts", GroupName="10. Discord", Order=0)]
public bool EnableDiscordAlerts { get; set; } = false;

[Display(Name="Discord Webhook URL", GroupName="10. Discord", Order=1,
    Description="Discord webhook URL (keep this private!)")]
public string DiscordWebhookUrl { get; set; } = "";

[Display(Name="Discord High Impact Only", GroupName="10. Discord", Order=2)]
public bool DiscordHighImpactOnly { get; set; } = true;

[Display(Name="Include Forecast Data", GroupName="10. Discord", Order=3)]
public bool DiscordIncludeForecastData { get; set; } = true;

[Display(Name="Mention Role (optional)", GroupName="10. Discord", Order=4,
    Description="Discord role ID to mention for High impact (e.g., @traders)")]
public string DiscordMentionRole { get; set; } = "";
```

---

## Implementation Details

### Core Alert Logic

```csharp
protected override void OnBarUpdate()
{
    // ... existing code ...

    // Process alerts if enabled
    if (EnableAlerts)
        ProcessAlerts(nowBar, relevant);
}

private void ProcessAlerts(DateTime currentTime, List<NewsEvent> events)
{
    // Parse alert windows
    var alertWindows = ParseAlertWindows(AlertWindowsCsv);

    foreach (var ev in events)
    {
        // Skip if impact level not enabled
        if (ev.Impact == "High" && !AlertHighImpact) continue;
        if (ev.Impact == "Medium" && !AlertMediumImpact) continue;

        // Calculate minutes until event
        double minutesUntil = (ev.LocalTime - currentTime).TotalMinutes;

        // Check if event is in the future
        if (minutesUntil <= 0) continue;

        // Check each alert window
        foreach (int windowMinutes in alertWindows)
        {
            // Check if we're in alert window (within tolerance)
            double tolerance = GetBarPeriodMinutes() * 1.5; // Allow some margin
            bool inAlertWindow = Math.Abs(minutesUntil - windowMinutes) <= tolerance;

            if (inAlertWindow)
            {
                // Check if already alerted
                if (!HasBeenAlerted(ev, windowMinutes))
                {
                    FireAlert(ev, windowMinutes, currentTime);
                    RecordAlert(ev, windowMinutes, currentTime);
                }
            }
        }
    }

    // Clean old alert history (older than 7 days)
    CleanAlertHistory();
}

private List<int> ParseAlertWindows(string csv)
{
    var windows = new List<int>();
    foreach (var part in (csv ?? "").Split(new[] { ',', ';', ' ' },
        StringSplitOptions.RemoveEmptyEntries))
    {
        if (int.TryParse(part.Trim(), out int minutes) && minutes > 0)
            windows.Add(minutes);
    }
    return windows.OrderByDescending(x => x).ToList(); // Largest first
}

private bool HasBeenAlerted(NewsEvent ev, int windowMinutes)
{
    return _alertHistory.Any(a =>
        a.EventTime == ev.LocalTime &&
        a.EventTitle == ev.Title &&
        a.AlertWindow == windowMinutes);
}

private void FireAlert(NewsEvent ev, int minutesBefore, DateTime currentTime)
{
    // Build alert message
    string message = FormatAlertMessage(ev, minutesBefore);

    // Generate unique alert name
    string alertName = $"{ev.LocalTime:yyyyMMddHHmm}_{ev.Impact}_{minutesBefore}min";

    // Determine color
    Brush bgColor = ev.Impact == "High" ? Brushes.Red : Brushes.Orange;

    // Fire NinjaTrader alert
    Alert(alertName,
          AlertPriority,
          message,
          AlertSoundFile,
          0,  // Don't rearm
          bgColor,
          Brushes.White);

    // Log to output
    Print($"[NewsAlert] {message}");

    // Fire Discord alert (if enabled)
    if (EnableDiscordAlerts && !string.IsNullOrEmpty(DiscordWebhookUrl))
    {
        Task.Run(() => SendDiscordAlertAsync(ev, minutesBefore));
    }
}

private string FormatAlertMessage(NewsEvent ev, int minutesBefore)
{
    var format = AlertMessageFormat
        .Replace("{IMPACT}", ev.Impact)
        .Replace("{TITLE}", ev.Title)
        .Replace("{CURRENCY}", ev.Currency ?? "")
        .Replace("{TIME}", ev.LocalTime.ToString("HH:mm"))
        .Replace("{MINUTES}", minutesBefore.ToString());

    return format;
}

private void RecordAlert(NewsEvent ev, int windowMinutes, DateTime alertedAt)
{
    _alertHistory.Add(new AlertRecord
    {
        EventTime = ev.LocalTime,
        EventTitle = ev.Title,
        Impact = ev.Impact,
        AlertWindow = windowMinutes,
        AlertedAt = alertedAt
    });
}

private void CleanAlertHistory()
{
    var cutoff = DateTime.Now.AddDays(-7);
    _alertHistory.RemoveAll(a => a.EventTime < cutoff);
}

private double GetBarPeriodMinutes()
{
    if (BarsPeriod.BarsPeriodType == BarsPeriodType.Minute)
        return BarsPeriod.Value;
    if (BarsPeriod.BarsPeriodType == BarsPeriodType.Second)
        return BarsPeriod.Value / 60.0;
    return 1.0; // Default
}
```

### Trading Hours Check

```csharp
private bool IsInTradingHours(DateTime time)
{
    if (!AlertOnlyDuringTradingHours)
        return true;

    if (Instrument?.TradingHours == null)
        return true; // If no trading hours defined, allow

    // Check if time is within instrument's trading hours
    var sessions = Instrument.TradingHours.GetTradingDayBeginEnd(
        time, BarsPeriod.BarsPeriodType);

    return time >= sessions.Begin && time <= sessions.End;
}
```

### Alert History Management

```csharp
private class AlertRecord
{
    public DateTime EventTime { get; set; }
    public string EventTitle { get; set; }
    public string Impact { get; set; }
    public int AlertWindow { get; set; }
    public DateTime AlertedAt { get; set; }
}

// Add to class fields
private readonly List<AlertRecord> _alertHistory = new List<AlertRecord>();

// Serialize/deserialize for persistence (optional)
protected override void OnStateChange()
{
    if (State == State.Terminated)
    {
        // Could save alert history to file if needed
        SaveAlertHistory();
    }

    if (State == State.DataLoaded)
    {
        // Could load alert history from file if needed
        LoadAlertHistory();
    }
}
```

---

## Discord Integration (Future)

### Discord Webhook Setup

1. **Create Webhook in Discord**:
   - Server Settings → Integrations → Webhooks
   - Create webhook, copy URL
   - Keep URL private (it's a secret!)

2. **Webhook URL Format**:
   ```
   https://discord.com/api/webhooks/{webhook.id}/{webhook.token}
   ```

### Implementation

```csharp
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
using Newtonsoft.Json; // Add reference to Newtonsoft.Json

private async Task SendDiscordAlertAsync(NewsEvent ev, int minutesBefore)
{
    try
    {
        using (var client = new HttpClient())
        {
            client.Timeout = TimeSpan.FromSeconds(5);

            // Build embed
            var embed = new
            {
                title = $"📢 {ev.Impact} Impact News Alert",
                description = ev.Title,
                color = GetDiscordColor(ev.Impact),
                fields = BuildDiscordFields(ev, minutesBefore),
                timestamp = DateTime.UtcNow.ToString("yyyy-MM-ddTHH:mm:ss.fffZ"),
                footer = new { text = "BadBuddha NewsMarkers PRO" }
            };

            // Add role mention if configured
            string content = "";
            if (ev.Impact == "High" && !string.IsNullOrEmpty(DiscordMentionRole))
            {
                content = $"<@&{DiscordMentionRole}>";
            }

            var payload = new
            {
                content = content,
                embeds = new[] { embed }
            };

            var json = JsonConvert.SerializeObject(payload);
            var httpContent = new StringContent(json, Encoding.UTF8, "application/json");

            var response = await client.PostAsync(DiscordWebhookUrl, httpContent);

            if (!response.IsSuccessStatusCode)
            {
                Print($"[Discord] Failed to send alert: {response.StatusCode}");
            }
        }
    }
    catch (Exception ex)
    {
        Print($"[Discord] Error sending alert: {ex.Message}");
    }
}

private object[] BuildDiscordFields(NewsEvent ev, int minutesBefore)
{
    var fields = new List<object>
    {
        new { name = "⏰ Time", value = ev.LocalTime.ToString("HH:mm"), inline = true },
        new { name = "💱 Currency", value = ev.Currency ?? "N/A", inline = true },
        new { name = "⚠️ Impact", value = ev.Impact, inline = true },
        new { name = "⏱️ Alert", value = $"{minutesBefore} min before", inline = true }
    };

    if (DiscordIncludeForecastData)
    {
        if (!string.IsNullOrEmpty(ev.Forecast))
            fields.Add(new { name = "📊 Forecast", value = ev.Forecast, inline = true });
        if (!string.IsNullOrEmpty(ev.Previous))
            fields.Add(new { name = "📈 Previous", value = ev.Previous, inline = true });
    }

    return fields.ToArray();
}

private int GetDiscordColor(string impact)
{
    // Discord colors in decimal (0xRRGGBB converted to decimal)
    switch (impact)
    {
        case "High":   return 0xE74C3C; // Red
        case "Medium": return 0xE67E22; // Orange
        case "Low":    return 0x3498DB; // Blue
        default:       return 0x95A5A6; // Gray
    }
}
```

### Discord Best Practices

1. **Rate Limiting**: Discord limits webhooks to ~5 messages per second
2. **Error Handling**: Wrap in try-catch, don't crash indicator
3. **Async Execution**: Use `Task.Run()` to avoid blocking chart
4. **Testing**: Test with Discord webhook testing tools first
5. **Security**: Never share webhook URL publicly

---

## Testing Strategy

### Phase 1: Audio Alerts

#### Unit Testing
- [ ] Alert window parsing (single, multiple, invalid)
- [ ] Alert deduplication logic
- [ ] Message formatting with placeholders
- [ ] Trading hours check

#### Integration Testing
- [ ] Test on 1-minute chart with 15min, 5min, 1min alerts
- [ ] Verify no duplicate alerts for same event
- [ ] Test High vs Medium filtering
- [ ] Test with AutoByInstrument filter
- [ ] Test with CustomCurrencyList filter
- [ ] Test alert sound plays correctly

#### Real-World Testing
- [ ] Run overnight with live data feed
- [ ] Monitor Output window for alert logs
- [ ] Verify alert timing accuracy
- [ ] Test with different bar periods (1min, 5min, 15min)

### Phase 2: Discord Alerts

#### Unit Testing
- [ ] Webhook URL validation
- [ ] JSON payload formatting
- [ ] Color code mapping
- [ ] Async execution doesn't block

#### Integration Testing
- [ ] Send test webhook to Discord
- [ ] Verify embed formatting
- [ ] Test with/without forecast data
- [ ] Test role mentions
- [ ] Test error handling (invalid webhook)

---

## Migration Path

### Step 1: Implement Core Alert Logic (Week 1)
1. Add alert properties to indicator
2. Implement `ProcessAlerts()` method
3. Add alert history tracking
4. Implement `FireAlert()` with NinjaTrader Alert()

### Step 2: Testing & Refinement (Week 2)
1. Test with live data
2. Adjust alert window tolerance
3. Fine-tune message formatting
4. Document alert configuration

### Step 3: Discord Integration (Week 3-4)
1. Add Discord properties
2. Implement `SendDiscordAlertAsync()`
3. Add JSON serialization
4. Test webhook integration
5. Document Discord setup process

---

## Configuration Examples

### Example 1: Conservative High Impact Trader
```
EnableAlerts = true
AlertHighImpact = true
AlertMediumImpact = false
AlertWindowsCsv = "15"
AlertSoundFile = "Alert2.wav"
AlertOnlyDuringTradingHours = true
```

### Example 2: Aggressive Multi-Alert Trader
```
EnableAlerts = true
AlertHighImpact = true
AlertMediumImpact = true
AlertWindowsCsv = "30,15,5,1"
AlertSoundFile = "Alert3.wav"
AlertOnlyDuringTradingHours = false
```

### Example 3: Discord-Only Silent Alerts
```
EnableAlerts = false
EnableDiscordAlerts = true
DiscordHighImpactOnly = true
DiscordIncludeForecastData = true
DiscordMentionRole = "123456789" // Your @traders role ID
```

---

## Security Considerations

### Discord Webhook Security
- ⚠️ **Never commit webhook URLs to version control**
- ⚠️ **Don't share workspace files with webhook URLs**
- ⚠️ **Use environment variables or encrypted storage**
- ⚠️ **Rotate webhooks if compromised**

### Alert Privacy
- Audio alerts may contain sensitive information
- Consider muting alerts during screen sharing
- Discord messages are visible to channel members

---

## Performance Considerations

1. **Alert History Growth**: Clean history older than 7 days
2. **Discord HTTP Calls**: Async, don't block chart rendering
3. **Alert Window Checks**: Only check upcoming events
4. **Bar Period Impact**: 1-min bars = more frequent checks

---

## Future Enhancements

### Possible V2 Features
- 📧 Email alerts via SMTP
- 📱 SMS alerts via Twilio
- 📊 Chart screenshot with alert
- 🔔 Push notifications to mobile app
- 📝 Alert log file export (CSV)
- 🎯 Custom alert conditions (e.g., "Only if market is open")
- 🔊 Text-to-speech alerts
- 🌐 Multi-language support

---

## Summary

This design provides:
- ✅ Flexible audio alert system with NinjaTrader integration
- ✅ Multiple alert windows per event
- ✅ Deduplication to prevent alert spam
- ✅ Respects existing filter logic
- ✅ Clean architecture for Discord integration
- ✅ Comprehensive configuration options
- ✅ Future-proof extensibility

**Next Steps**:
1. Review and approve design
2. Begin Phase 1 implementation (audio alerts)
3. Test with live data
4. Plan Phase 2 (Discord) based on user feedback

---

**Document Version**: 1.0
**Last Updated**: October 2025
**Author**: BadBuddha Customs Development Team
