# Date/Time Handling: Where Sanity Goes to Die

## What Seems Simple
"Just show the user what time something happened."

## The Actual Complexity

### 1. The JavaScript Date Constructor of Doom
```javascript
// These all create DIFFERENT dates:
new Date('2024-01-01');        // UTC midnight
new Date('2024-01-01T00:00');  // Local midnight
new Date('2024/01/01');        // Local midnight
new Date('01-01-2024');        // May parse as MM-DD or DD-MM
new Date('January 1, 2024');   // Local midnight
new Date('1/1/24');            // 1924 or 2024? Depends on browser!

// The cursed parsing
new Date('2024-01-01').getTime() !== new Date('2024-01-01T00:00').getTime()
// TRUE! Different times despite looking identical
```

### 2. Daylight Saving Time Violence
```javascript
// Spring forward - 2am becomes 3am
const march10 = new Date('2024-03-10 02:30:00');
// This time literally doesn't exist!
// Some browsers: Invalid Date
// Others: 03:30:00
// Others: 01:30:00

// Fall back - 2am happens twice
const nov3_first = new Date('2024-11-03 01:30:00 PST');
const nov3_second = new Date('2024-11-03 01:30:00 PDT');
// Same wall clock time, one hour apart!
// How do you store "1:30am on Nov 3"?
```

### 3. The Timezone Database Changes RETROACTIVELY
```javascript
// Morocco decided in 2018 to stay on DST permanently
// But they changed their mind in 2019
// Then changed AGAIN during Ramadan

// Your stored timestamp: '2019-05-05 14:00:00 Africa/Casablanca'
// Displayed in 2018: "3:00 PM"
// Displayed in 2019: "2:00 PM"
// Same stored time, different display!

// Turkey eliminated DST in 2016
// All historical times from 2015 now display differently
```

### 4. The Leap Second Conspiracy
```javascript
// December 31, 2016 had 86,401 seconds (leap second)
const leapMoment = new Date('2016-12-31T23:59:60Z');
// JavaScript: "Invalid Date" (pretends it doesn't exist)

// But it DID exist!
// Systems that acknowledge it: Different timestamps
// Systems that ignore it: Off by 1 second
// Your uptime counter: Now wrong
```

### 5. The 24:00 vs 00:00 Problem
```javascript
// Japanese trains run until "25:00" (1 AM next day)
// Business hours: "Open until 24:00"
// Is that:
new Date('2024-01-01 24:00:00'); // Invalid
new Date('2024-01-02 00:00:00'); // Next day
// Same moment, different mental model
```

### 6. Historical Date Disasters
```javascript
// Russia skipped Feb 1-13, 1918 (calendar change)
const russianFeb = new Date('1918-02-05');
// This date never happened in Russia!

// JavaScript thinks it did
// Your "age of document" calculation: Wrong by 13 days

// Sweden had February 30th in 1712
new Date('1712-02-30'); // JavaScript: March 1
// Historically accurate? Nope.
```

### 7. The Millisecond Precision Lie
```javascript
Date.now(); // 1704067200000

// Looks precise to millisecond!
// Reality on various systems:
// - Windows: ~15.6ms resolution (updates 64 times/second)
// - Some browsers: Reduced to 100ms for fingerprinting protection
// - With Spectre mitigations: Intentionally fuzzy

performance.now(); // Higher precision
// But also lies for security!
```

### 8. Timezone Abbreviation Hell
```javascript
// "CST" means:
// - Central Standard Time (UTC-6)
// - China Standard Time (UTC+8)
// - Cuba Standard Time (UTC-5)

new Date('2024-01-01 12:00 CST');
// Which CST? Browser guesses!
// Could be 14 hours off
```

### 9. The Birthday Problem
```javascript
// User born on Feb 29, 2000 (leap year)
const birth = new Date('2000-02-29');

// Calculate age on Feb 28, 2024:
const age = Math.floor((today - birth) / 365.25 / 86400 / 1000);
// Wrong! Doesn't account for leap years correctly

// When do they celebrate in non-leap years?
// Feb 28? March 1? Legally varies by country!
```

### 10. The Server-Client Time Drift
```javascript
// Server: 2024-01-01 12:00:00.000
// Client clock is 3 seconds fast
// Message timestamp: "3 seconds ago"
// But message appears to be from the future!

// User sees: "-3 seconds ago"
// Or: "in 3 seconds"
// Or: app crashes on negative duration
```

### 11. The Relative Time Formatter Chaos
```javascript
const rtf = new Intl.RelativeTimeFormat('en');
rtf.format(-1, 'day'); // "1 day ago"

// But when exactly does "yesterday" become "2 days ago"?
// Midnight? 24 hours? Depends on locale!

// "Last week" in some cultures: Previous calendar week
// Others: 7 days ago
// Business context: Last 5 business days
```

### 12. The Storage Encoding Nightmare
```javascript
// Store as:
// 1. Unix timestamp (loses timezone)
// 2. ISO 8601 string (parsing ambiguity)
// 3. Separate fields (reconstruction errors)
// 4. With timezone (timezone rules change)
// 5. As UTC (loses user intent)

// User sets alarm for 9 AM daily
// Store as UTC: 14:00 (for EST)
// DST changes: Alarm now at 8 AM or 10 AM!
// User confused and angry
```

### 13. Duration vs Period Insanity
```javascript
// Add "one month" to January 31
const jan31 = new Date('2024-01-31');
jan31.setMonth(jan31.getMonth() + 1);
// February 31 doesn't exist!
// Result: March 2 or 3 (depends on leap year)

// Add "one year" to Feb 29
const feb29 = new Date('2020-02-29');
feb29.setFullYear(feb29.getFullYear() + 1);
// Result: March 1, 2021

// "Monthly" subscription started Jan 31
// When is next billing?
// Feb 28? Mar 1? Mar 3?
```

### 14. The Infinite Timezone List
```javascript
// There are ~600 timezone identifiers
// But wait! Historical timezones: ~1000s more
// Military zones: 25 more
// Offset variations: ±15 minutes, ±30, ±45

// Nepal: UTC+5:45 (yes, 45 minutes!)
// Newfoundland: UTC-3:30
// Lord Howe Island: UTC+10:30 (UTC+11 in DST)
// But DST there is only 30 minutes, not 1 hour!
```

## Real-World Disasters

### The Y2K38 Problem
```javascript
// Unix timestamp uses 32-bit signed integer
// Maxes out at: January 19, 2038 03:14:07 UTC
const doomsday = new Date(2147483647 * 1000);

// After that, time wraps to 1901!
// Your mortgage calculator: Suddenly thinks it's 1901
// Your SSL certificates: All invalid
// Your database: Complete chaos
```

### The Microsoft Excel 1900 Bug
```javascript
// Excel thinks 1900 was a leap year (it wasn't)
// Excel date 60 = February 29, 1900 (doesn't exist)
// For compatibility, this bug is now a "feature"
// Affects all Excel exports/imports on the web
```

## The Ultimate Solution That Doesn't Exist

```javascript
// What we want:
const perfect = new MagicalDate('2024-01-01 12:00 America/New_York');
perfect.add({ months: 1, days: 15 });
perfect.toUserString(); // Respects user preferences
perfect.isBusinessHour(); // Knows about holidays
perfect.getDiff(otherDate); // Handles all edge cases

// What we have:
// - Date() - Broken since 1995
// - Moment.js - 67kb, deprecated
// - date-fns - Still can't handle timezones properly
// - Temporal API - Proposed for 5+ years, still not shipped
// - Luxon, Day.js, etc. - Each with their own bugs

// Every app: Implements its own broken date logic
```

## Why This Will Never Be Fixed
Time is political. Countries change timezones for economic reasons, wars, or just because. There's no universal truth about what time it is, was, or will be. We're all just pretending we know what time means.