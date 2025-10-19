# 📅 Schedule Examples & CSV Templates

## Overview

This folder contains schedule examples and the **CSV template** that makes scheduling human-friendly! 

**Why CSV?** Because editing 252 YAML entries is painful, but editing a spreadsheet is easy! 📊

---

## 📂 Files in This Folder

### 1. `heating_schedule_template.csv` ⭐

**The magic file!** Your complete heating schedule in table format.

**What's inside:**
- 4 rooms (Living, Birou, Dormitor, Cam Luca)
- 7 days (Monday-Sunday)
- 9 time slots per day per room
- 252 total schedule entries
- Easy to read and edit in Excel/LibreOffice

**Open with:**
- Microsoft Excel
- LibreOffice Calc
- Google Sheets (upload first)
- Any CSV editor

---

## 📊 CSV Structure

### Table Layout

```csv
Area,Monday,,Tuesday,,Wednesday,,Thursday,,Friday,,Saturday,,Sunday,
,Time interval,Temperature,Time interval,Temperature,Time interval,Temperature,...

Living,00:00 - 06:30,20,00:00 - 06:30,20,00:00 - 06:30,20,...
,06:30 - 08:00,22,06:30 - 08:00,22,06:30 - 08:00,22,...
,08:00 - 13:30,20,08:00 - 13:30,20,08:00 - 13:30,20,...
```

### Column Structure

**For each day (2 columns):**
1. **Time interval** → "HH:MM - HH:MM" format
2. **Temperature** → Target temperature in °C

**Example row:**
```csv
Living,06:30 - 08:00,22,06:30 - 08:00,22,06:30 - 08:00,22,...
       ^^^^^^^^^^^^  ^^  (Monday)         (Tuesday)
       Time          Temp
```

---

## 🎯 Living Room Schedule (Example)

### Weekdays (Monday-Friday)

| Time | Temp | Purpose |
|------|------|---------|
| 00:00 - 06:30 | 20°C | Sleeping (eco) |
| 06:30 - 08:00 | 22°C | Morning routine (comfort) |
| 08:00 - 13:30 | 20°C | Away at work (eco) |
| 13:30 - 16:00 | 20°C | Still away (eco) |
| 16:00 - 17:00 | 20°C | Preheating start |
| 17:00 - 19:00 | 22°C | Evening home (comfort) |
| 19:00 - 21:30 | 22°C | Family time (comfort) |
| 21:30 - 23:00 | 20°C | Bedtime prep (eco) |
| 23:00 - 00:00 | 20°C | Night (eco) |

**Logic:**
- **Comfort zones:** When family is home
- **Eco zones:** When away or sleeping
- **2°C difference:** Balance comfort vs energy

---

### Weekend (Saturday-Sunday)

| Time | Temp | Purpose |
|------|------|---------|
| 00:00 - 06:30 | 20°C | Sleeping |
| 06:30 - 08:00 | 20°C | Lazy morning (no rush) |
| 08:00 - 13:30 | 22°C | Home all day (comfort) |
| 13:30 - 16:00 | 22°C | Relaxing (comfort) |
| 16:00 - 17:00 | 22°C | Still home (comfort) |
| 17:00 - 19:00 | 22°C | Evening (comfort) |
| 19:00 - 21:30 | 22°C | Family time (comfort) |
| 21:30 - 23:00 | 22°C | Extended comfort |
| 23:00 - 00:00 | 20°C | Bedtime |

**Logic:**
- **Extended comfort:** Home all day
- **No morning rush:** Stay at 20°C until 08:00
- **More hours @ 22°C:** Weekend relaxation

---

## 🏢 Office (Birou) Schedule

### All Days Pattern

**Weekdays:** High evening comfort (work-from-home evenings)

| Time | Temp |
|------|------|
| 00:00 - 06:30 | 20°C |
| 06:30 - 08:00 | 22°C |
| 08:00 - 13:30 | 20°C |
| 13:30 - 17:00 | 20°C |
| 17:00 - 23:00 | 22°C |
| 23:00 - 00:00 | 22°C |

**Weekend:** All-day comfort (home office)

| Time | Temp |
|------|------|
| 00:00 - 06:30 | 20°C |
| 06:30 - 00:00 | 22°C |

**Logic:**
- Evening priority (17:00-23:00)
- Weekend = extended comfort
- Night economy

---

## 🛏️ Bedroom (Dormitor) Schedule

### Stable Temperature Approach

**All days similar:**

| Time | Temp | Note |
|------|------|------|
| 00:00 - 06:30 | 20°C | Sleeping temperature |
| 06:30 - 08:00 | 22°C | Morning warmth (weekdays) |
| 06:30 - 08:00 | 21°C | Gentle warmth (weekend) |
| 08:00 - 13:30 | 20/21°C | Lower during day |
| Rest of day | 20°C | Stable eco |

**Logic:**
- **Sleep quality first:** Stable 20°C for sleeping
- **Morning boost:** Brief warmth when waking
- **Weekend difference:** 1°C lower (slower morning)
- **Simplicity:** No complex patterns

---

## 👶 Child's Room (Cam Luca) Schedule

### Variable by Day

**More complex pattern with different temps per day!**

**Monday example:**
| Time | Temp |
|------|------|
| 00:00 - 06:30 | 20°C |
| 06:30 - 08:00 | 22°C |
| 08:00 - 13:30 | 20°C |
| 17:00 - 19:00 | 20°C |
| 19:00 - 21:30 | 20°C |
| 21:30 - 23:00 | 22°C |
| 23:00 - 00:00 | 20°C |

**Tuesday example:**
| Time | Temp | Note |
|------|------|------|
| 16:00 - 17:00 | 22°C | After school activity |
| 17:00 - 19:00 | 20°C | Homework time |
| 19:00 - 21:30 | 21°C | Dinner + family |
| 21:30 - 23:00 | 22°C | Bedtime comfort |

**Logic:**
- **Activity-based:** Different temps for different activities
- **School schedule:** Adjusted by weekday
- **Bedtime priority:** 21:30-23:00 always comfortable
- **Weekend freedom:** Extended comfort hours

---

## 🔧 How to Edit CSV

### Step 1: Open in Excel/LibreOffice

```bash
# Double-click file:
heating_schedule_template.csv

# Or right-click → Open With → Excel
```

### Step 2: View as Table

**You'll see:**
- Row 1: Headers (Area, Monday, Tuesday, etc.)
- Row 2: Column types (Time interval, Temperature)
- Row 3+: Actual schedules

**Tips:**
- Freeze first 2 rows (View → Freeze Panes)
- Enable filters (Data → Filter)
- Zoom to comfortable level

### Step 3: Edit Temperatures

**Simply change numbers in temperature columns!**

**Before:**
```csv
Living,06:30 - 08:00,22,...
```

**After (warmer mornings):**
```csv
Living,06:30 - 08:00,23,...
```

### Step 4: Edit Time Intervals

**Format: "HH:MM - HH:MM"**

**Before:**
```csv
Living,06:30 - 08:00,22,...
```

**After (earlier wake):**
```csv
Living,06:00 - 07:30,22,...
```

⚠️ **Important:** Keep the space before and after the dash!

### Step 5: Save File

```bash
File → Save
# Keep CSV format (not Excel .xlsx)
```

---

## 🔄 Converting CSV to YAML

### Manual Method

**For each time interval in CSV:**

```csv
Living,06:30 - 08:00,22  (Monday)
```

**Becomes in YAML:**

```yaml
- time: "06:30:00"
  weekdays: [1]  # Monday = 1
  min_temp: 22
```

### Weekday Numbers

```yaml
Monday    = 1
Tuesday   = 2
Wednesday = 3
Thursday  = 4
Friday    = 5
Saturday  = 6
Sunday    = 7
```

### Full Room Example

**CSV rows:**
```csv
Living,00:00 - 06:30,20,00:00 - 06:30,20  (Mon-Tue)
Living,06:30 - 08:00,22,06:30 - 08:00,22  (Mon-Tue)
```

**YAML equivalent:**
```yaml
- platform: climate_scheduler
  name: Living Room Heating
  profiles:
    - id: "Weekly Schedule"
      schedule:
        # Monday
        - time: "00:00:00"
          weekdays: [1]
          min_temp: 20
        - time: "06:30:00"
          weekdays: [1]
          min_temp: 22
        
        # Tuesday
        - time: "00:00:00"
          weekdays: [2]
          min_temp: 20
        - time: "06:30:00"
          weekdays: [2]
          min_temp: 22
```

---

## 💡 Quick Editing Tips

### Tip 1: Copy Pattern Across Days

**Same schedule Mon-Fri?**
1. Enter Monday's temps
2. Select Monday column (temperatures only)
3. Copy (Ctrl+C)
4. Select Tue-Fri temp columns
5. Paste (Ctrl+V)

**Boom! 5 days done instantly.** ⚡

---

### Tip 2: Weekend Pattern

**Usually simpler than weekdays:**
1. Keep 20°C for sleeping
2. Extend 22°C for home hours
3. Make Sat/Sun identical

**Example:**
```csv
Sat: 06:30-23:00 @ 22°C (all comfort)
Sun: 06:30-23:00 @ 22°C (same)
```

---

### Tip 3: Find & Replace

**Lower all temps by 1°C?**

```
Find:    ,22,
Replace: ,21,
```

**Raise bedroom temps?**

```
Find:    Dormitor,...,20,
Replace: Dormitor,...,21,
```

---

### Tip 4: Conditional Formatting

**Visualize temperatures:**

1. Select all temperature columns
2. Format → Conditional Formatting
3. Color scale: Blue (15°C) → Red (25°C)
4. See patterns instantly! 🌡️

**Result:**
- Cold = Blue
- Warm = Red
- Spot outliers easily

---

### Tip 5: Add Notes Column

**Track your reasoning:**

```csv
Area,Mon Time,Mon Temp,Notes
Living,06:30-08:00,22,Morning coffee time
Living,08:00-13:30,20,Everyone at work/school
```

**Helps you remember why you chose those times!**

---

## 📋 Validation Checklist

Before converting CSV → YAML, verify:

### Time Intervals

- ✅ Format: "HH:MM - HH:MM" (with spaces)
- ✅ Start time < End time
- ✅ 00:00 included for each day
- ✅ Covers full 24 hours
- ✅ No gaps between intervals

### Temperatures

- ✅ Realistic range (15-25°C)
- ✅ No decimals if not needed
- ✅ Consistent across similar days
- ✅ Minimum safe temp (≥15°C away mode)

### Structure

- ✅ Each room has all 7 days
- ✅ Same number of intervals per day
- ✅ Area names match your rooms
- ✅ No empty cells in data rows

---

## 🎯 Sample Scenarios

### Scenario 1: Simple Weekday/Weekend

**Weekday:** Work from 9-5
**Weekend:** Home all day

**Living Room CSV:**
```csv
Mon-Fri: 00:00-07:00 @ 20°C
Mon-Fri: 07:00-09:00 @ 22°C
Mon-Fri: 09:00-17:00 @ 20°C
Mon-Fri: 17:00-23:00 @ 22°C
Mon-Fri: 23:00-00:00 @ 20°C

Sat-Sun: 00:00-08:00 @ 20°C
Sat-Sun: 08:00-23:00 @ 22°C
Sat-Sun: 23:00-00:00 @ 20°C
```

**Simple! Only 2 patterns.** ✅

---

### Scenario 2: Complex Child Schedule

**Mon/Wed/Fri:** Sports after school
**Tue/Thu:** Homework heavy
**Weekend:** Flexible

**Cam Luca CSV:**
```csv
Mon: 15:00-17:00 @ 20°C (sports away)
Mon: 17:00-19:00 @ 23°C (warm after sports)

Tue: 15:00-18:00 @ 22°C (homework comfort)
Tue: 18:00-19:00 @ 21°C (cool down)

Weekend: 08:00-21:00 @ 22°C (comfort all day)
```

**Different pattern each day!** 🎯

---

### Scenario 3: Office Worker Schedule

**Mon-Thu:** Office 9-6
**Fri:** Work from home
**Weekend:** Errands Saturday, rest Sunday

**Living Room CSV:**
```csv
Mon-Thu: 09:00-18:00 @ 20°C (away)
Fri: 09:00-18:00 @ 22°C (home office)
Sat: Variable (errands)
Sun: 08:00-22:00 @ 22°C (rest day)
```

**Friday = different!** 📅

---

## 🔍 Troubleshooting CSV

### Issue: Excel converts times to dates

**Problem:**
```csv
06:30 - 08:00  →  Excel shows: 6/30/2024
```

**Fix:**
1. Select time columns
2. Format Cells → Text
3. Re-enter times

**Or:**
- Add apostrophe: `'06:30 - 08:00`
- Excel treats as text

---

### Issue: Commas in temperature field

**Problem:**
```csv
Living,06:30,22,00  ← Too many commas!
```

**Fix:**
- Temperature should be single number
- No commas in number (European format: 22,5 → use 22.5)

---

### Issue: Missing midnight entry

**Problem:**
```csv
Living,06:30-08:00,22,...  ← Where's 00:00?
```

**Fix:**
- ALWAYS start each day with 00:00 entry
- Required for climate_scheduler

---

### Issue: Time overlap

**Problem:**
```csv
Living,08:00-13:00,20
Living,12:00-17:00,22  ← Overlap!
```

**Fix:**
- Adjust end time of first interval
- Or start time of second interval
- No overlaps allowed

---

## 📚 Full Schedule Example

**Let's build a complete room from scratch!**

### Requirements

- Bedroom
- Mon-Fri: Work 9-5
- Weekend: Sleep late
- Cool sleeping temps

### CSV Result

```csv
Area,Monday,,Tuesday,,Wednesday,,Thursday,,Friday,,Saturday,,Sunday,
,Time,Temp,Time,Temp,Time,Temp,Time,Temp,Time,Temp,Time,Temp,Time,Temp

Dormitor,00:00-07:00,19,00:00-07:00,19,00:00-07:00,19,00:00-07:00,19,00:00-07:00,19,00:00-09:00,19,00:00-09:00,19
,07:00-08:00,21,07:00-08:00,21,07:00-08:00,21,07:00-08:00,21,07:00-08:00,21,09:00-10:00,20,09:00-10:00,20
,08:00-22:00,19,08:00-22:00,19,08:00-22:00,19,08:00-22:00,19,08:00-23:00,19,10:00-23:00,20,10:00-23:00,20
,22:00-00:00,19,22:00-00:00,19,22:00-00:00,19,22:00-00:00,19,23:00-00:00,19,23:00-00:00,19,23:00-00:00,19
```

**Key points:**
- 19°C for sleeping (cool)
- 21°C brief morning warmth (weekdays)
- 20°C weekend mornings (gentler)
- Sleep later on weekends (09:00 vs 07:00)
- Friday night extended to 23:00

---

## 🎓 Learning Path

### Beginner: 1 Room, 2 Patterns

**Start with:**
- Living room only
- Weekday pattern (Mon-Fri same)
- Weekend pattern (Sat-Sun same)
- 3-4 time slots per day

**Example:**
```csv
Mon-Fri: 00:00-08:00 @ 20, 08:00-17:00 @ 20, 17:00-23:00 @ 22, 23:00-00:00 @ 20
Sat-Sun: 00:00-08:00 @ 20, 08:00-23:00 @ 22, 23:00-00:00 @ 20
```

**Time:** 15 minutes ⏱️

---

### Intermediate: 2 Rooms, 3 Patterns

**Add:**
- Bedroom (different schedule)
- More time slots (6-7 per day)
- Sunday different from Saturday

**Example:**
```csv
Living: Mon-Fri pattern, Sat pattern, Sun pattern
Bedroom: Simpler pattern (sleep focus)
```

**Time:** 30 minutes ⏱️

---

### Advanced: 4+ Rooms, Unique Days

**Full system:**
- All rooms
- Each day potentially different
- Activity-based timing
- Seasonal variations

**Like our example:** 252 entries! 🎯

**Time:** 1-2 hours ⏱️

---

## 💪 Pro Editing Techniques

### Technique 1: Use Formulas

**Auto-calculate end times:**

```excel
A1: 06:30  (start)
B1: 08:00  (end)

C1: =B1  (next start)
D1: 13:30  (next end)
```

**No gaps, no overlaps!** ✅

---

### Technique 2: Data Validation

**Prevent invalid temperatures:**

1. Select temp columns
2. Data → Data Validation
3. Allow: Whole number
4. Minimum: 15
5. Maximum: 25

**Now Excel blocks 30°C entry!** 🚫

---

### Technique 3: VLOOKUP Profiles

**Reference table:**

```excel
Profile  | Morning | Day | Evening | Night
Comfort  | 22      | 22  | 22      | 20
Eco      | 20      | 19  | 20      | 19
Boost    | 23      | 22  | 23      | 21
```

**Then use VLOOKUP to fill schedule!**

**Benefit:** Change profile, all temps update instantly! ⚡

---

### Technique 4: Named Ranges

**Name your temp columns:**
```
LivingMonTemp = B:B
LivingTueTemp = D:D
```

**Then reference in formulas:**
```excel
=AVERAGE(LivingMonTemp)  (average Mon temp)
```

**Cleaner formulas!** 📊

---

## 🎯 Common Patterns Library

### Pattern 1: Basic Weekday

```csv
00:00-07:00 @ 20°C  (sleeping)
07:00-09:00 @ 22°C  (morning)
09:00-17:00 @ 20°C  (away)
17:00-23:00 @ 22°C  (evening)
23:00-00:00 @ 20°C  (night)
```

---

### Pattern 2: Weekend Comfort

```csv
00:00-08:00 @ 20°C  (sleeping late)
08:00-23:00 @ 22°C  (home all day)
23:00-00:00 @ 20°C  (bedtime)
```

---

### Pattern 3: Night Shift Worker

```csv
00:00-08:00 @ 20°C  (sleeping)
08:00-16:00 @ 22°C  (awake at home)
16:00-23:00 @ 20°C  (away at work)
23:00-00:00 @ 22°C  (coming home)
```

---

### Pattern 4: Bedroom Minimal

```csv
00:00-07:00 @ 19°C  (sleeping cool)
07:00-08:00 @ 21°C  (morning warmth)
08:00-00:00 @ 19°C  (eco rest of day)
```

---

### Pattern 5: Office Remote Work

```csv
00:00-09:00 @ 20°C  (morning prep)
09:00-12:00 @ 22°C  (work session)
12:00-13:00 @ 21°C  (lunch break)
13:00-18:00 @ 22°C  (afternoon work)
18:00-00:00 @ 20°C  (evening eco)
```

---

## ✅ Export Checklist

Before converting to YAML, verify CSV:

### Structure
- ✅ Header row present
- ✅ Area names correct
- ✅ All 7 days included
- ✅ Time/Temp columns paired

### Data Quality
- ✅ No empty cells
- ✅ All temps in range (15-25°C)
- ✅ Times in "HH:MM - HH:MM" format
- ✅ Each day starts with 00:00

### Coverage
- ✅ All rooms included
- ✅ Full 24h covered each day
- ✅ No time gaps
- ✅ No time overlaps

### Logic
- ✅ Patterns make sense
- ✅ Realistic for lifestyle
- ✅ Comfortable temps when home
- ✅ Eco temps when away

---

## 🎁 Bonus: Seasonal Templates

### Winter Template (Dec-Feb)

**Higher comfort temps:**
```csv
Comfort: 23°C (was 22°C)
Eco: 21°C (was 20°C)
```

---

### Spring/Fall Template (Mar-May, Sep-Nov)

**Standard temps:**
```csv
Comfort: 22°C
Eco: 20°C
```

---

### Summer Template (Jun-Aug)

**Lower or off:**
```csv
Option 1: Comfort 20°C, Eco 18°C
Option 2: Turn off heating completely
```

---

## 🚀 Next Steps

1. **Open CSV** in Excel/LibreOffice
2. **Edit temps** for your lifestyle
3. **Save changes**
4. **Convert to YAML** (see complete_heating_schedulers.yaml)
5. **Deploy to HA**
6. **Test and refine**

---

## 📞 Related Files

- **Complete YAML version:** See `../three-profile-system/complete_heating_schedulers.yaml`
- **Deployment guide:** See `../DEPLOYMENT_GUIDE.md`
- **Main README:** See `../README.md`

---

**Happy scheduling! 📅🔥**
