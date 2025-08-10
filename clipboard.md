# Clipboard

## Java

```
import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.WeekFields;

import java.awt.Toolkit;
import java.awt.datatransfer.StringSelection;
import java.awt.GraphicsEnvironment;
import java.awt.HeadlessException;

public class time_stamp {
    public static void main(String[] args) {
        ZonedDateTime now = ZonedDateTime.now();
        ZoneId tz = now.getZone();

        // 3-digit numeric fields by prefixing a literal 0 to 2-digit tokens
        String year = now.format(DateTimeFormatter.ofPattern("yyyy"));
        String month = now.format(DateTimeFormatter.ofPattern("0MM")); // e.g., 007
        String day = now.format(DateTimeFormatter.ofPattern("0dd"));   // e.g., 004
        String hour = now.format(DateTimeFormatter.ofPattern("0HH"));
        String minute = now.format(DateTimeFormatter.ofPattern("0mm"));
        String second = now.format(DateTimeFormatter.ofPattern("0ss"));

        // Nanoseconds
        String nano = String.format("%09d", now.getNano());

        // ISO week/year/day
        WeekFields wf = WeekFields.ISO;
        int isoYear = now.get(wf.weekBasedYear());
        int isoWeek = now.get(wf.weekOfWeekBasedYear());
        int isoDOW = now.get(wf.dayOfWeek());

        // Day-of-year (3-digit)
        String doy = String.format("%03d", now.getDayOfYear());

        // TZ id with _slash_ instead of /
        String tzId = tz.getId().replace("/", "_slash_");

        // Unix timestamp with nanoseconds as decimal seconds
        // Unix timestamp seconds and nanoseconds separately, joined by underscore
        long unix_seconds = now.toEpochSecond();
        int nanos = now.getNano();
        String unix_timestamp_string = String.format("%d_%09d", unix_seconds, nanos);

        // Build underscore string:
        // YYYY_MMM_DDD_HHH_MMM_SSS_NNNNNNNNN_TimeZone_ISOYEAR_WWWW_WEEKDAY_YYYY_DOY_UnixSeconds_Nanoseconds
        String output = String.format(
                "%s_%s_%s_%s_%s_%s_%s_%s_%04d_W%03d_%03d_%s_%s_%s",
                year, month, day, hour, minute, second, nano, tzId,
                isoYear, isoWeek, isoDOW, year, doy, unix_timestamp_string);

        // Print result
        System.out.println(output);

        // Copy to clipboard
        copy_to_clipboard(output);
    }

    public static void copy_to_clipboard(String text) {
        try {
            if (!GraphicsEnvironment.isHeadless()) {
                StringSelection selection = new StringSelection(text);
                Toolkit.getDefaultToolkit().getSystemClipboard().setContents(selection, null);
            }
        } catch (HeadlessException e) {
            // Silent fail
        }
    }
}
```

## Python

```
"""
Date/time utility functions with clipboard copy
"""

from datetime import datetime
from zoneinfo import ZoneInfo
import tzlocal
import time
import pyperclip

def get_timestamp() -> str:
    """
    Returns a high-precision, time zone-aware timestamp string:
    YYYY_MMM_DDD_HHH_MMM_SSS_NNNNNNNNN_TimeZone_ISOYEAR_WWWW_WEEKDAY_YYYY_DOY_UnixSeconds_Nanoseconds

    Example:
        2025_008_004_013_048_029_083107000_America_slash_New_York_2025_W032_001_2025_216_1754681668_083107000
    """
    # Local TZ and now with nanoseconds
    local_tz: ZoneInfo = tzlocal.get_localzone()
    ns_since_epoch: int = time.time_ns()
    seconds_since_epoch: int = ns_since_epoch // 1_000_000_000  # integer seconds
    nanos_part: int = ns_since_epoch % 1_000_000_000
    now: datetime = datetime.fromtimestamp(seconds_since_epoch, tz=ZoneInfo(local_tz.key))

    # Components
    year: str = f"{now.year}"
    month: str = f"{now.month:03d}"
    day: str = f"{now.day:03d}"
    hour: str = f"{now.hour:03d}"
    minute: str = f"{now.minute:03d}"
    second: str = f"{now.second:03d}"
    nanoseconds: str = f"{nanos_part:09d}"
    time_zone: str = local_tz.key.replace("/", "_slash_")

    # ISO week info and ordinal day
    iso_year, iso_week, iso_weekday = now.isocalendar()
    iso_week_str: str = f"{iso_week:03d}"
    iso_weekday_str: str = f"{iso_weekday:03d}"
    day_of_year: str = f"{now.timetuple().tm_yday:03d}"

    return (
        f"{year}_{month}_{day}_{hour}_{minute}_{second}_"
        f"{nanoseconds}_{time_zone}_{iso_year:04d}_W{iso_week_str}_{iso_weekday_str}_"
        f"{year}_{day_of_year}_{seconds_since_epoch}_{nanoseconds}"
    )


if __name__ == "__main__":
    timestamp = get_timestamp()
    print(timestamp)

    # Copy to clipboard silently
    pyperclip.copy(timestamp)
```

## Powershell
```
2025_008_010_014_015_017_070057900_America_slash_New_York_2025_W032_007_2025_222_1754849717_070057900_192_168_004_042_LANDINGCOMPUTER_LANDINGCOMPUTER_backslash_peter_Powershell_7_5_2
C:\GitHub-repositories\time-stuff\powershell> Get-UnderscoreTimestamp
2025_008_010_014_015_020_203258100_America_slash_New_York_2025_W032_007_2025_222_1754849720_203258100
2025_008_010_014_015_058_441091100_America_slash_New_York_2025_W032_007_2025_222_1754849758_441091100_192_168_004_042_LANDINGCOMPUTER_LANDINGCOMPUTER_backslash_peter_Powershell_7_5_2
C:\GitHub-repositories\time-stuff\powershell> Get-UnderscoreTimestamp | Set-Clipboard  
2025_008_010_014_016_003_536783800_America_slash_New_York_2025_W032_007_2025_222_1754849763_536783800_192_168_004_042_LANDINGCOMPUTER_LANDINGCOMPUTER_backslash_peter_Powershell_7_5_2
C:\GitHub-repositories\time-stuff\powershell> Get-UnderscoreTimestamp | Tee-Object -Variable ts | Set-Clipboard; $ts
2025_008_010_014_016_030_050866300_America_slash_New_York_2025_W032_007_2025_222_1754849790_050866300
2025_008_010_014_016_030_065980200_America_slash_New_York_2025_W032_007_2025_222_1754849790_065980200_192_168_004_042_LANDINGCOMPUTER_LANDINGCOMPUTER_backslash_peter_Powershell_7_5_2
C:\GitHub-repositories\time-stuff\powershell>
```

```
2025_008_010_014_014_052_724979100_America_slash_New_York_2025_W032_007_2025_222_1754849692_724979100_192_168_004_042_LANDINGCOMPUTER_LANDINGCOMPUTER_backslash_peter_Powershell_7_5_2
C:\GitHub-repositories\time-stuff\go> cd ..\ ; ls

    Directory: C:\GitHub-repositories\time-stuff

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d----     2025-008-008 019.012.041                go
d----     2025-008-008 019.012.008                java
d----     2025-008-008 021.019.007                powershell
d----     2025-008-008 019.012.015                python
-a---     2025-008-008 018.014.049             10 README.md

2025_008_010_014_014_053_835880700_America_slash_New_York_2025_W032_007_2025_222_1754849693_835880700_192_168_004_042_LANDINGCOMPUTER_LANDINGCOMPUTER_backslash_peter_Powershell_7_5_2
C:\GitHub-repositories\time-stuff> cd .\powershell\
2025_008_010_014_014_057_545638800_America_slash_New_York_2025_W032_007_2025_222_1754849697_545638800_192_168_004_042_LANDINGCOMPUTER_LANDINGCOMPUTER_backslash_peter_Powershell_7_5_2
C:\GitHub-repositories\time-stuff\powershell> ls

    Directory: C:\GitHub-repositories\time-stuff\powershell

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---     2025-008-008 021.019.019             53 f

2025_008_010_014_014_058_024530100_America_slash_New_York_2025_W032_007_2025_222_1754849698_024530100_192_168_004_042_LANDINGCOMPUTER_LANDINGCOMPUTER_backslash_peter_Powershell_7_5_2
C:\GitHub-repositories\time-stuff\powershell> (Get-Command Get-UnderscoreTimestamp).ScriptBlock

<#
.SYNOPSIS
Generates an underscore-delimited, timezone-aware, nanosecond-precision timestamp string.

.DESCRIPTION
`Get-UnderscoreTimestamp` produces a precise timestamp string in the format:
YYYY_MMM_DDD_HHH_MMM_SSS_NNNNNNNNN_TimeZone_ISOYEAR_WWWW_WEEKDAY_YYYY_DOY_UnixSeconds_Nanoseconds

Field breakdown:
- Year (YYYY)
- Month (MMM) — 3 digits by prefixing a literal 0 to a 2-digit month (e.g., January -> 001 becomes 001? actually 0MM yields 001–012; August -> 008)
- Day of month (DDD) — 3 digits by prefixing a literal 0 to a 2-digit day (e.g., 009, 031)
- Hour of day (HHH, 000–023)
- Minute (MMM, 000–059)
- Second (SSS, 000–059)
- Nanoseconds (NNNNNNNNN, 9 digits)
- IANA timezone identifier with slashes replaced by `_slash_` (e.g., America_slash_New_York)
- ISO year (4 digits)
- ISO week (Wnnn, e.g., W032)
- ISO day of week (001–007, Monday = 001)
- Calendar year (YYYY)
- Day of year (DOY, 001–366)
- Unix epoch seconds (integer)
- Nanoseconds (again, 9 digits)

.PARAMETER Date
The date/time to format. Defaults to the current system date/time.

.EXAMPLE
PS> Get-UnderscoreTimestamp
2025_008_009_020_051_026_177317200_America_slash_New_York_2025_W032_006_2025_221_1754787086_177317200

.EXAMPLE
PS> Get-UnderscoreTimestamp -Date (Get-Date "2025-08-08T14:23:45.1234567Z")
2025_008_008_014_023_045_123456700_Coordinated_slash_Universal_2025_W032_005_2025_221_1754663025_123456700

.NOTES
- Uses `Get-IanaTimeZone` for IANA timezone resolution and replaces "/" with "_slash_".
- Uses `Get-IsoWeekDate` for ISO year/week/day calculations (returned as yyyy-Wwww-ddd).
- Nanoseconds are derived from .NET ticks within the second: (`$Date.Ticks % 10,000,000`) * 100.
#>

    [CmdletBinding()]
    param(
        [datetime]$Date = (Get-Date)
    )

    # Year
    $year = $Date.Year

    # Month (zero-padded to 3 digits by prefixing a literal 0)
    $month = ('0{0:D2}' -f $Date.Month)

    # Day of month (zero-padded to 3 digits by prefixing a literal 0)
    $day = ('0{0:D2}' -f $Date.Day)

    # Time components (zero-padded to 3)
    $hour   = '{0:000}' -f $Date.Hour
    $minute = '{0:000}' -f $Date.Minute
    $second = '{0:000}' -f $Date.Second

    # Nanoseconds from ticks (ticks are 100ns)
    $ticksWithinSecond = $Date.Ticks % 10000000
    $nanoseconds = '{0:000000000}' -f ($ticksWithinSecond * 100)

    # IANA timezone with _slash_ replacement
    $iana = Get-IanaTimeZone
    $tz_formatted = $iana -replace '/', '_slash_'

    # ISO year-week-weekday
    $iso = Get-IsoWeekDate -date $Date
    if ($iso -notmatch '^(?<y>\d{4})-W(?<w>\d{3})-(?<d>\d{3})$') {
        throw "Unexpected ISO week format: $iso"
    }
    $iso_year = $Matches['y']
    $iso_week = $Matches['w']
    $iso_dow  = $Matches['d']

    # Day of year (3-digit)
    $doy = '{0:D3}' -f $Date.DayOfYear

    # Unix seconds
    $unixSeconds = [DateTimeOffset]$Date
    $unixSeconds = $unixSeconds.ToUnixTimeSeconds()

    # Final string
    return "$year`_${month}`_${day}`_${hour}`_${minute}`_${second}`_${nanoseconds}`_${tz_formatted}`_${iso_year}_W${iso_week}_$iso_dow`_${year}_$doy`_${unixSeconds}_$nanoseconds"

2025_008_010_014_015_017_070057900_America_slash_New_York_2025_W032_007_2025_222_1754849717_070057900_192_168_004_042_LANDINGCOMPUTER_LANDINGCOMPUTER_backslash_peter_Powershell_7_5_2
C:\GitHub-repositories\time-stuff\powershell> Get-UnderscoreTimestamp
2025_008_010_014_015_020_203258100_America_slash_New_York_2025_W032_007_2025_222_1754849720_203258100
2025_008_010_014_015_058_441091100_America_slash_New_York_2025_W032_007_2025_222_1754849758_441091100_192_168_004_042_LANDINGCOMPUTER_LANDINGCOMPUTER_backslash_peter_Powershell_7_5_2
C:\GitHub-repositories\time-stuff\powershell> Get-UnderscoreTimestamp | Set-Clipboard  
2025_008_010_014_016_003_536783800_America_slash_New_York_2025_W032_007_2025_222_1754849763_536783800_192_168_004_042_LANDINGCOMPUTER_LANDINGCOMPUTER_backslash_peter_Powershell_7_5_2
C:\GitHub-repositories\time-stuff\powershell> Get-UnderscoreTimestamp | Tee-Object -Variable ts | Set-Clipboard; $ts
2025_008_010_014_016_030_050866300_America_slash_New_York_2025_W032_007_2025_222_1754849790_050866300
2025_008_010_014_016_030_065980200_America_slash_New_York_2025_W032_007_2025_222_1754849790_065980200_192_168_004_042_LANDINGCOMPUTER_LANDINGCOMPUTER_backslash_peter_Powershell_7_5_2
C:\GitHub-repositories\time-stuff\powershell>
```

## Go

```
package main

import (
    "fmt"
    "log"

    "github.com/atotto/clipboard"
    "github.com/PeterCullenBurbery/go_functions_002/v5/date_time_functions"
)

func main() {
    timestamp, err := date_time_functions.Get_timestamp()
    if err != nil {
        log.Fatalf("Failed to get timestamp: %v", err)
    }

    fmt.Println(timestamp)

    // Copy to clipboard using atotto/clipboard package
    if err := clipboard.WriteAll(timestamp); err != nil {
        log.Fatalf("Failed to copy to clipboard: %v", err)
    }
}
```