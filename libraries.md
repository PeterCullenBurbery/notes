Libraries
    Powershell libraries
        1. Powershell functions007 at https://www.powershellgallery.com/packages/PowershellFunctions007
    Python libraries
        1. peter_cullen_burbery_python_functions at https://pypi.org/project/peter-cullen-burbery-python-functions/
    Go libraries
        1. go_functions_002
    Bash libraries

functions
## Get-UnderscoreTimestamp (Powershell)
```
function Get-UnderscoreTimestamp {
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
}
```

## get_timestamp (Python)

```
from datetime import datetime
from zoneinfo import ZoneInfo
import tzlocal
import time

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
```

Get_timestamp (Go)