# NewsMarkers PRO Alert System - Quick Reference

## Overview

Complete alerting system for BadBuddha NewsMarkers PRO indicator with:
- 🔔 **Audio alerts** using NinjaTrader's built-in Alert() method
- 💬 **Discord webhooks** for team notifications (future)
- ⚙️ **Flexible configuration** with multiple alert windows
- 🎯 **Smart deduplication** to prevent alert spam

---

## Quick Start

### 1. Enable Audio Alerts (5 minutes)

```
Properties → 09. Alerts
├─ Enable Alerts: true
├─ Alert High Impact Events: true
├─ Alert Medium Impact Events: false
└─ Alert Windows: "15,5"
```

**Result**: Alerts at 15 min and 5 min before High impact events

### 2. Test Configuration

1. Use **1-minute chart** for best accuracy
2. Enable **EnableDebug = true** to see alert processing
3. Watch **Output Window** (Ctrl+F2) for `[NewsAlert]` messages
4. Wait for next event in your alert window

### 3. Verify It Works

- ✅ Alert sound plays
- ✅ Alert window pops up in NinjaTrader
- ✅ `[NewsAlert]` message in Output window
- ✅ No duplicate alerts for same event

---

## Configuration Presets

### Preset 1: Conservative (High Impact Only)
```yaml
EnableAlerts: true
AlertHighImpact: true
AlertMediumImpact: false
AlertWindowsCsv: "15"
AlertSoundFile: "Alert2.wav"
AlertPriority: High
AlertOnlyDuringTradingHours: true
AlertMessageFormat: "{IMPACT} in {MINUTES}min: {TITLE} ({CURRENCY})"
```
**Use Case**: Day traders focused on major news only

---

### Preset 2: Aggressive (Multi-Alert)
```yaml
EnableAlerts: true
AlertHighImpact: true
AlertMediumImpact: true
AlertWindowsCsv: "30,15,5,1"
AlertSoundFile: "Alert3.wav"
AlertPriority: High
AlertOnlyDuringTradingHours: false
```
**Use Case**: Active traders who want multiple warnings

---

### Preset 3: Quiet (Minimal)
```yaml
EnableAlerts: true
AlertHighImpact: true
AlertMediumImpact: false
AlertWindowsCsv: "5"
AlertSoundFile: "Alert1.wav"
AlertPriority: Medium
```
**Use Case**: Traders who want just a quick heads-up

---

## Property Reference

### Core Alert Settings

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `EnableAlerts` | bool | false | Master on/off switch |
| `AlertHighImpact` | bool | true | Alert for High impact events |
| `AlertMediumImpact` | bool | false | Alert for Medium impact events |
| `AlertWindowsCsv` | string | "15,5" | Minutes before event (comma-separated) |

### Alert Behavior

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `AlertSoundFile` | string | "Alert2.wav" | Sound file name or full path |
| `AlertPriority` | Priority | High | NinjaTrader priority level |
| `AlertOnlyDuringTradingHours` | bool | false | Respect trading hours |
| `AlertMessageFormat` | string | (see below) | Custom message template |

**Default Message Format**:
```
"{IMPACT} Impact in {MINUTES} min: {TITLE} ({CURRENCY} {TIME})"
```

**Available Placeholders**:
- `{IMPACT}` - High, Medium, Low
- `{TITLE}` - Event title
- `{CURRENCY}` - Currency code (USD, EUR, etc.)
- `{TIME}` - Event time (HH:mm format)
- `{MINUTES}` - Minutes until event

---

## Alert Windows Explained

### Single Window
```
AlertWindowsCsv: "15"
```
**Result**: One alert 15 minutes before event

### Multiple Windows
```
AlertWindowsCsv: "30,15,5,1"
```
**Result**: Four alerts (30min, 15min, 5min, 1min before)

### Timing Tolerance
- Alert fires when current time is within **1.5 bar periods** of window
- Example: On 1-min chart, window is ±1.5 minutes
- Example: On 5-min chart, window is ±7.5 minutes

**Recommendation**: Use **1-minute charts** for precise timing

---

## Alert Sounds

### Built-in NinjaTrader Sounds
```
"Alert1.wav"
"Alert2.wav"
"Alert3.wav"
"Alert4.wav"
```
Located in: `C:\Program Files\NinjaTrader 8\sounds\`

### Custom Sound Files
```
AlertSoundFile: "C:\\Users\\YourName\\Documents\\MyAlert.wav"
```
**Requirements**:
- Must be `.wav` format
- Full path required
- Test sound plays in Windows first

---

## Discord Integration

### Quick Setup (Phase 2)

1. **Create Webhook**:
   - Discord Server → Channel → Edit Channel
   - Integrations → Webhooks → New Webhook
   - Copy webhook URL

2. **Configure Indicator**:
   ```yaml
   EnableDiscordAlerts: true
   DiscordWebhookUrl: "https://discord.com/api/webhooks/..."
   DiscordHighImpactOnly: true
   DiscordIncludeForecastData: true
   ```

3. **Test**:
   - Wait for next alert
   - Check Discord channel
   - Check Output window for `[Discord]` logs

### Discord Message Preview
```
🚨 High Impact News Alert
Non-Farm Payrolls (USD)
──────────────────────
⏰ Time: 08:30
💱 Currency: USD
⚠️ Impact: High
⏱️ Alert Window: 15 min before
📊 Forecast: 200K
📈 Previous: 185K
──────────────────────
BadBuddha NewsMarkers PRO • ES 03-25
```

---

## Alert Filtering

Alerts respect all existing filter settings:

### Instrument Filter
```
EventFilter: AutoByInstrument
```
**Result**: Only alerts for events matching your instrument

Example: Trading **ES** → Only **USD** alerts

### Custom Currency Filter
```
EventFilter: CustomCurrencyList
CustomCurrenciesCsv: "USD,EUR"
```
**Result**: Only USD and EUR alerts

### Impact Filter
```
AlertHighImpact: true
AlertMediumImpact: false
```
**Result**: Only High impact alerts (Medium filtered out)

**✅ Alerts are automatically filtered just like chart display**

---

## Troubleshooting

### Problem: No Alerts Firing

**Check**:
- [ ] `EnableAlerts = true`?
- [ ] Are there upcoming events? (check Events Table)
- [ ] Is current time within alert window?
- [ ] Enable `EnableDebug = true` and check Output
- [ ] Are events filtered out by currency filter?

**Debug Output Example**:
```
[NewsPRO] Alert fired: News_202510090830_High_15min - High Impact in 15 min: NFP (USD 08:30)
```

---

### Problem: Duplicate Alerts

**Cause**: Should not happen (built-in deduplication)

**Check**:
- [ ] Same event firing multiple times?
- [ ] Different instruments in same workspace?
- [ ] Check Output window for duplicate `[NewsAlert]` messages

**Solution**: Each alert is tracked in history to prevent duplicates

---

### Problem: Alert Sound Not Playing

**Check**:
- [ ] Sound file exists?
- [ ] Path correct? (test file in Windows Media Player)
- [ ] Volume on?
- [ ] NinjaTrader sound enabled? (Tools → Options → Sounds)

**Test**:
```csharp
AlertSoundFile: "Alert2.wav"  // Try built-in first
```

---

### Problem: Timing Off by Several Minutes

**Cause**: Chart bar period too large

**Solution**:
- Use **1-minute chart** for best accuracy
- Alerts fire on `OnBarUpdate()` (bar close)
- 5-min bars = alerts can be off by up to 5 minutes

---

## Best Practices

### ✅ Do's

- ✅ Use 1-minute charts for accurate timing
- ✅ Start with single alert window (e.g., "15") for testing
- ✅ Enable debug mode during setup
- ✅ Test with small number of alert windows first
- ✅ Keep webhook URLs private (Discord)
- ✅ Use trading hours filter for day trading

### ❌ Don'ts

- ❌ Don't use 15-min+ bars if you want precise timing
- ❌ Don't set dozens of alert windows (alert spam)
- ❌ Don't share Discord webhook URLs
- ❌ Don't expect sub-bar accuracy (alerts fire on bar close)
- ❌ Don't alert for every Low impact event (too noisy)

---

## Performance Impact

### Alert System Overhead
- **Minimal**: Checks only happen on `OnBarUpdate()`
- **Efficient**: Only checks upcoming events in date range
- **Async**: Discord calls don't block chart rendering

### Memory Usage
- Alert history stored in memory
- Auto-cleanup: Removes alerts older than 7 days
- Typical usage: < 100 KB

### Recommendation
- No performance concerns for normal use
- Works fine on multiple charts simultaneously

---

## Testing Checklist

Before going live:

### Audio Alerts
- [ ] Alerts fire at correct times
- [ ] No duplicate alerts
- [ ] Alert sound audible
- [ ] Message format correct
- [ ] Respects impact filters (High/Medium)
- [ ] Respects currency filters
- [ ] Trading hours filter works (if enabled)
- [ ] Multiple alert windows work
- [ ] Alert history cleanup works

### Discord Integration (Phase 2)
- [ ] Webhook URL configured
- [ ] Test webhook sends message
- [ ] Embed formatting correct
- [ ] Forecast data included (if enabled)
- [ ] Role mentions work (if configured)
- [ ] Error handling works (try invalid webhook)
- [ ] Doesn't block chart rendering

---

## Configuration Examples by Trading Style

### Scalper (ES/NQ)
```yaml
AlertWindowsCsv: "5,1"
AlertHighImpact: true
AlertMediumImpact: false
AlertOnlyDuringTradingHours: true
EventFilter: AutoByInstrument
```
**Why**: Quick heads-up right before major news

---

### Swing Trader (Multi-Market)
```yaml
AlertWindowsCsv: "30,15"
AlertHighImpact: true
AlertMediumImpact: true
AlertOnlyDuringTradingHours: false
EventFilter: CustomCurrencyList
CustomCurrenciesCsv: "USD,EUR,GBP,JPY"
```
**Why**: Early warning for major and medium impact across currencies

---

### News Trader (Volatility Hunter)
```yaml
AlertWindowsCsv: "15,5,2"
AlertHighImpact: true
AlertMediumImpact: false
AlertSoundFile: "Alert3.wav"
AlertPriority: High
EnableDiscordAlerts: true
```
**Why**: Multiple alerts with Discord for team coordination

---

## Support

### Debug Mode
```
EnableDebug: true
```
Prints detailed info to Output window:
- Alert processing
- Event filtering
- Alert firing
- Discord sends
- Error messages

### Output Window Messages
```
[NewsAlert] High Impact in 15 min: NFP (USD 08:30)
[Discord] Alert sent: Non-Farm Payrolls
[NewsPRO] Alert fired: News_202510090830_High_15min
```

### Common Debug Messages
| Message | Meaning |
|---------|---------|
| `Alert fired` | Alert successfully triggered |
| `Recorded alert` | Alert tracked (prevents duplicates) |
| `Cleaned alert history` | Old alerts removed |
| `NoAlertWindows` | Invalid AlertWindowsCsv format |
| `[Discord] Alert sent` | Discord webhook success |
| `[Discord][ERROR]` | Discord webhook failed |

---

## Summary

### What You Get

✅ **Audio Alerts**: Built-in NinjaTrader alerts with sound
✅ **Multiple Windows**: Alert at 30min, 15min, 5min, 1min (configurable)
✅ **Smart Filtering**: Respects all existing filters
✅ **No Duplicates**: Built-in deduplication
✅ **Discord Ready**: Easy webhook integration (Phase 2)
✅ **Flexible Config**: Customize messages, sounds, timing
✅ **Trading Hours**: Optional respect for market hours
✅ **Debug Mode**: Detailed logging for troubleshooting

### Implementation Effort

| Phase | Effort | Time |
|-------|--------|------|
| **Phase 1: Audio** | Easy | 30 min |
| **Phase 2: Discord** | Moderate | 1-2 hours |

### Files Included

1. `NewsMarkers_Alert_System_Design.md` - Complete design document
2. `NewsMarkers_Alert_Implementation.cs` - Audio alert code
3. `NewsMarkers_Discord_Integration.cs` - Discord webhook code
4. `Alert_System_Quick_Reference.md` - This file

---

**Ready to implement?** Start with `NewsMarkers_Alert_Implementation.cs`

**Questions?** Enable debug mode and check Output window

**Need help?** Check the troubleshooting section above

---

*Document Version: 1.0 | October 2025*
