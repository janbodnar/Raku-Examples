# DateTime

DateTime handling in Raku provides comprehensive support for working with  
dates, times, durations, and time zones. Raku's temporal types include  
DateTime for complete timestamp representation, Date for calendar dates,  
Instant for precise moments in time, and Duration for time spans. These  
types work together to provide powerful date and time manipulation  
capabilities with proper time zone support, arithmetic operations, and  
flexible formatting options. This chapter covers everything from basic  
DateTime creation to advanced temporal calculations and best practices.  

## Current date and time

Creating DateTime objects representing the current moment.  

```raku
my $now = DateTime.now;
say $now;

my $utc_now = DateTime.now(utc => True);
say $utc_now;

say "Current time: {DateTime.now}";
say "UTC time: {DateTime.now(:utc)}";
```

The `DateTime.now` method creates a DateTime object for the current moment.  
Use `:utc` to get the current UTC time instead of local time. Both  
methods return precise timestamps including microseconds.  

## Creating specific dates

Constructing DateTime objects for specific dates and times.  

```raku
my $birthday = DateTime.new(
    year => 1990,
    month => 5,
    day => 15,
    hour => 14,
    minute => 30,
    second => 0
);
say $birthday;

my $simple = DateTime.new(
    year => 2024,
    month => 12,
    day => 25
);
say $simple;

my $with_timezone = DateTime.new(
    year => 2024,
    month => 7,
    day => 4,
    hour => 16,
    minute => 0,
    timezone => -14400  # -4 hours in seconds
);
say $with_timezone;
```

DateTime.new accepts year, month, day, hour, minute, second, and timezone  
parameters. Missing time components default to 0, timezone defaults to  
local. Timezone is specified in seconds offset from UTC.  

## Date objects

Working with Date objects for calendar dates without time.  

```raku
my $today = Date.today;
say $today;

my $specific_date = Date.new(2024, 3, 15);
say $specific_date;

my $from_string = Date.new('2024-12-31');
say $from_string;

say "Today's day of year: {$today.day-of-year}";
say "Days in this month: {$today.days-in-month}";
say "Is leap year: {$today.is-leap-year}";
```

Date objects represent calendar dates without time information. Create  
them with Date.today for current date, Date.new with year/month/day,  
or from ISO date strings. Date objects provide calendar properties.  

## Instant objects

Working with Instant objects for precise time measurements.  

```raku
my $instant = Instant.now;
say $instant;

my $epoch = Instant.from-posix(0);
say $epoch;

my $future = Instant.from-posix(2000000000);
say $future;

say "Seconds since epoch: {$instant.to-posix}";
say "Instant type: {$instant.WHAT}";
```

Instant represents a precise moment in time as seconds since the Unix  
epoch. Use Instant.now for current moment, from-posix to create from  
timestamp, and to-posix to convert back to timestamp.  

## Duration arithmetic

Performing calculations with Duration objects.  

```raku
my $start = DateTime.now;
my $duration = Duration.new(3600);  # 1 hour in seconds
my $end = $start + $duration;

say "Start: $start";
say "Duration: $duration seconds";
say "End: $end";

my $longer = Duration.new(days => 2, hours => 3, minutes => 30);
say "Complex duration: $longer";

my $later = DateTime.now + Duration.new(weeks => 1);
say "One week from now: $later";
```

Duration objects represent time spans. Create them with seconds or  
named parameters (days, hours, minutes, seconds). Add Duration to  
DateTime objects to get new DateTime objects.  

## Date arithmetic with integers

Simple date arithmetic using integer days.  

```raku
my $today = Date.today;
my $yesterday = $today - 1;
my $tomorrow = $today + 1;
my $next_week = $today + 7;

say "Yesterday: $yesterday";
say "Today: $today";
say "Tomorrow: $tomorrow";
say "Next week: $next_week";

my $past_date = Date.new(2024, 1, 15);
my $days_diff = $today - $past_date;
say "Days since Jan 15, 2024: $days_diff";
```

Date objects support arithmetic with integers representing days. Adding  
or subtracting integers moves the date forward or backward. Subtracting  
two dates gives the number of days between them.  

## DateTime components

Accessing individual components of DateTime objects.  

```raku
my $dt = DateTime.new(
    year => 2024,
    month => 8,
    day => 15,
    hour => 14,
    minute => 30,
    second => 45,
    timezone => 7200  # +2 hours
);

say "Year: {$dt.year}";
say "Month: {$dt.month}";
say "Day: {$dt.day}";
say "Hour: {$dt.hour}";
say "Minute: {$dt.minute}";
say "Second: {$dt.second}";
say "Timezone: {$dt.timezone}";
say "Day of week: {$dt.day-of-week}";
say "Day of year: {$dt.day-of-year}";
```

DateTime objects provide accessor methods for all components. Timezone  
is in seconds offset from UTC. Day-of-week returns 1-7 (Monday=1),  
day-of-year returns 1-366.  

## String formatting

Converting DateTime objects to formatted strings.  

```raku
my $dt = DateTime.new(2024, 8, 15, 14, 30, 45);

say $dt.Str;
say $dt.gist;
say $dt.raku;

# ISO 8601 format
say $dt.iso8601;

# Custom formatting using strftime
say $dt.strftime('%Y-%m-%d %H:%M:%S');
say $dt.strftime('%A, %B %d, %Y');
say $dt.strftime('%I:%M %p');

# Date-only formatting
my $date = Date.today;
say $date.strftime('%d/%m/%Y');
```

Multiple formatting options available: Str for basic format, iso8601  
for ISO standard, strftime for custom patterns using standard format  
codes like %Y (year), %m (month), %d (day), %H (hour).  

## Parsing date strings

Converting strings to DateTime and Date objects.  

```raku
# ISO 8601 parsing
my $dt1 = DateTime.new('2024-08-15T14:30:45Z');
say $dt1;

my $dt2 = DateTime.new('2024-08-15T14:30:45+02:00');
say $dt2;

# Date parsing
my $date1 = Date.new('2024-08-15');
say $date1;

# Custom parsing would need additional logic
my $custom = '15/08/2024';
my @parts = $custom.split('/');
my $parsed = Date.new(@parts[2].Int, @parts[1].Int, @parts[0].Int);
say $parsed;
```

DateTime.new and Date.new can parse ISO 8601 formatted strings directly.  
For custom formats, split the string and extract components manually,  
then construct the objects with individual parameters.  

## Time zones

Working with different time zones and conversions.  

```raku
my $utc = DateTime.now(:utc);
say "UTC: $utc";

my $local = DateTime.now;
say "Local: $local";

# Convert to different timezone (offset in seconds)
my $pst = $utc.in-timezone(-28800);  # PST is UTC-8
say "PST: $pst";

my $cet = $utc.in-timezone(3600);    # CET is UTC+1
say "CET: $cet";

# Create with specific timezone
my $tokyo = DateTime.new(
    year => 2024, month => 8, day => 15,
    hour => 14, minute => 30,
    timezone => 32400  # JST is UTC+9
);
say "Tokyo time: $tokyo";
```

Time zones are represented as second offsets from UTC. Use in-timezone  
method to convert DateTime to different zones. Negative values are  
west of UTC, positive values are east of UTC.  

## Comparing dates

Comparing DateTime and Date objects for ordering.  

```raku
my $date1 = Date.new(2024, 8, 15);
my $date2 = Date.new(2024, 8, 20);
my $date3 = Date.new(2024, 8, 15);

say $date1 < $date2;     # True
say $date1 > $date2;     # False
say $date1 == $date3;    # True
say $date1 eq $date3;    # True

my $dt1 = DateTime.new(2024, 8, 15, 10, 30);
my $dt2 = DateTime.new(2024, 8, 15, 14, 30);

say $dt1 before $dt2;    # True
say $dt1 after $dt2;     # False
say $dt1 == $dt2;        # False
```

Date and DateTime objects support standard comparison operators.  
Use <, >, == for numerical comparison, eq for string equality,  
before/after methods for temporal ordering.  

## Working with weekdays

Determining and working with days of the week.  

```raku
my $today = Date.today;

say "Today is day {$today.day-of-week} of the week";
say "Today is a {$today.day-name}";

# Find next Monday
my $days_until_monday = (8 - $today.day-of-week) % 7;
$days_until_monday = 7 if $days_until_monday == 0;
my $next_monday = $today + $days_until_monday;
say "Next Monday: $next_monday";

# Check if today is weekend
my $is_weekend = $today.day-of-week >= 6;
say "Is weekend: $is_weekend";

# Get all weekdays in current week
my $monday = $today - ($today.day-of-week - 1);
for 0..6 -> $offset {
    my $day = $monday + $offset;
    say "{$day.day-name}: $day";
}
```

Day-of-week returns 1-7 (Monday=1, Sunday=7). Day-name returns the  
English name. Calculate offsets to find specific weekdays or iterate  
through week ranges.  

## Month calculations

Working with months and month boundaries.  

```raku
my $date = Date.new(2024, 8, 15);

say "Month: {$date.month}";
say "Month name: {$date.month-name}";
say "Days in month: {$date.days-in-month}";

# First and last day of month
my $first_of_month = Date.new($date.year, $date.month, 1);
my $last_of_month = Date.new($date.year, $date.month, $date.days-in-month);

say "First of month: $first_of_month";
say "Last of month: $last_of_month";

# Next month (handle year rollover)
my $next_month = $date.month == 12 
    ?? Date.new($date.year + 1, 1, 1)
    !! Date.new($date.year, $date.month + 1, 1);
say "First of next month: $next_month";
```

Use month-name for English month names, days-in-month for month length.  
Handle month arithmetic carefully to account for different month lengths  
and year boundaries.  

## Year calculations

Working with years and leap year handling.  

```raku
my $date = Date.new(2024, 2, 29);  # Leap year date

say "Year: {$date.year}";
say "Is leap year: {$date.is-leap-year}";
say "Days in year: {$date.days-in-year}";

# Check different years
for 2020..2025 -> $year {
    my $test_date = Date.new($year, 1, 1);
    say "Year $year is leap: {$test_date.is-leap-year}";
}

# Find next leap year
my $current_year = Date.today.year;
my $next_leap = $current_year + 1;
while !Date.new($next_leap, 1, 1).is-leap-year {
    $next_leap++;
}
say "Next leap year after $current_year: $next_leap";
```

Use is-leap-year to check for leap years, days-in-year returns 365 or  
366. Leap years occur every 4 years, except century years that aren't  
divisible by 400.  

## Date ranges and sequences

Creating sequences and ranges of dates.  

```raku
my $start = Date.new(2024, 8, 1);
my $end = Date.new(2024, 8, 7);

# Date range
my @week = $start..$end;
say @week;

# Using sequence operator
my @sequence = $start, $start + 1 ... $end;
say @sequence;

# Every other day
my @every_other = $start, $start + 2 ... $end;
say @every_other;

# All Mondays in August 2024
my $first_monday = Date.new(2024, 8, 5);  # First Monday
my @mondays = $first_monday, $first_monday + 7 ... Date.new(2024, 8, 31);
say @mondays;

# Date iteration
for $start..$end -> $date {
    say "Processing date: $date ({$date.day-name})";
}
```

Use range operator .. to create date ranges, sequence operator ... for  
custom progressions. Iterate through date ranges with for loops or  
create arrays for batch processing.  

## Time calculations

Calculating elapsed time and durations.  

```raku
my $start_time = DateTime.now;

# Simulate some work
sleep 1;

my $end_time = DateTime.now;
my $elapsed = $end_time - $start_time;

say "Start: $start_time";
say "End: $end_time";
say "Elapsed: $elapsed seconds";

# Convert duration to different units
my $duration = Duration.new(7265);  # 2 hours, 1 minute, 5 seconds
my $hours = $duration.Int div 3600;
my $minutes = ($duration.Int % 3600) div 60;
my $seconds = $duration.Int % 60;

say "Duration: {$hours}h {$minutes}m {$seconds}s";

# Age calculation
my $birth_date = Date.new(1990, 5, 15);
my $today = Date.today;
my $age_days = $today - $birth_date;
my $age_years = $age_days div 365.25;  # Approximate
say "Age: approximately {$age_years.Int} years ({$age_days} days)";
```

Subtracting DateTime objects returns Duration in seconds. Convert to  
hours/minutes/seconds using integer division. For age calculations,  
divide days by average year length.  

## Working with epochs

Converting between different epoch representations.  

```raku
my $now = DateTime.now;
my $posix_time = $now.posix;
my $instant = $now.Instant;

say "Current DateTime: $now";
say "POSIX timestamp: $posix_time";
say "As Instant: $instant";

# Convert from POSIX timestamp
my $from_posix = DateTime.from-posix(1692097845);
say "From POSIX: $from_posix";

# Convert from Instant
my $instant_obj = Instant.from-posix(1692097845);
my $from_instant = DateTime.new($instant_obj);
say "From Instant: $from_instant";

# Millisecond timestamp (JavaScript-style)
my $js_timestamp = $now.posix * 1000;
say "JavaScript timestamp: {$js_timestamp.Int}";
my $from_js = DateTime.from-posix($js_timestamp / 1000);
say "From JS timestamp: $from_js";
```

Use posix method to get Unix timestamp, from-posix to create DateTime  
from timestamp. Convert between different timestamp formats by  
multiplying/dividing appropriately.  

## Date validation

Validating and handling invalid dates.  

```raku
# Valid date creation
try {
    my $valid = Date.new(2024, 2, 29);  # Valid leap year date
    say "Valid date: $valid";
    CATCH {
        default { say "Invalid date: {.message}" }
    }
}

# Invalid date creation
try {
    my $invalid = Date.new(2023, 2, 29);  # Invalid: 2023 not leap year
    say "This won't print";
    CATCH {
        default { say "Caught invalid date: {.message}" }
    }
}

# Date validation function
sub is-valid-date($year, $month, $day) {
    try {
        Date.new($year, $month, $day);
        return True;
    }
    return False;
}

say "2024-02-29 valid: {is-valid-date(2024, 2, 29)}";
say "2023-02-29 valid: {is-valid-date(2023, 2, 29)}";
say "2024-13-01 valid: {is-valid-date(2024, 13, 1)}";
```

DateTime and Date constructors throw exceptions for invalid dates.  
Use try/CATCH blocks to handle validation errors gracefully. Create  
validation functions for user input.  

## Calendar calculations

Advanced calendar operations and business date logic.  

```raku
my $date = Date.today;

# Find the nth weekday of a month
sub nth-weekday($year, $month, $weekday, $n) {
    my $first = Date.new($year, $month, 1);
    my $first_weekday = $first.day-of-week;
    my $offset = ($weekday - $first_weekday) % 7;
    return $first + $offset + ($n - 1) * 7;
}

# Third Thursday of current month
my $third_thursday = nth-weekday($date.year, $date.month, 4, 3);
say "Third Thursday: $third_thursday";

# Business days (excluding weekends)
sub add-business-days($date, $days) {
    my $result = $date;
    my $added = 0;
    
    while $added < $days {
        $result++;
        if $result.day-of-week <= 5 {  # Monday-Friday
            $added++;
        }
    }
    return $result;
}

my $business_date = add-business-days($date, 10);
say "10 business days from today: $business_date";
```

Create helper functions for common calendar calculations like finding  
the nth occurrence of a weekday or calculating business days excluding  
weekends.  

## Time period checks

Checking if dates fall within specific periods.  

```raku
my $date = Date.today;

# Check if date is in current month
sub is-current-month($test_date) {
    my $today = Date.today;
    return $test_date.year == $today.year && $test_date.month == $today.month;
}

# Check if date is in current year
sub is-current-year($test_date) {
    return $test_date.year == Date.today.year;
}

# Check if date is within last N days
sub is-within-days($test_date, $days) {
    my $today = Date.today;
    my $diff = $today - $test_date;
    return 0 <= $diff <= $days;
}

say "Today is in current month: {is-current-month($date)}";
say "Date is in current year: {is-current-year($date)}";
say "Date is within 30 days: {is-within-days($date, 30)}";

# Age-based checks
my $birth_date = Date.new(2000, 6, 15);
my $age = (Date.today - $birth_date) / 365.25;
say "Person is adult: {$age >= 18}";
```

Create utility functions to check if dates fall within specific periods.  
Use date arithmetic to calculate differences and compare against  
thresholds.  

## Performance considerations

Best practices for DateTime performance and memory usage.  

```raku
# Reuse DateTime objects when possible
my $base_date = Date.today;
my @dates = gather for 1..1000 {
    take $base_date + $_;
}

say "Created {@dates.elems} dates";

# Use Instant for high-precision timing
my $start = Instant.now;
# ... some operation ...
my $end = Instant.now;
my $precise_duration = $end - $start;
say "Precise timing: $precise_duration";

# Cache frequently used dates
my %date_cache;
sub cached-date($year, $month, $day) {
    my $key = "$year-$month-$day";
    return %date_cache{$key} //= Date.new($year, $month, $day);
}

# Use appropriate types for your needs
my Date $just_date = Date.today;           # For date-only data
my DateTime $full_timestamp = DateTime.now; # For precise timestamps
my Instant $precise_moment = Instant.now;   # For performance timing
```

Choose appropriate temporal types for your use case. Cache frequently  
used dates, use Instant for precise timing measurements, and prefer  
Date over DateTime when time information isn't needed.  

## Date formatting patterns

Common date formatting patterns and localization considerations.  

```raku
my $dt = DateTime.new(2024, 8, 15, 14, 30, 45);

# Common US formats
say $dt.strftime('%m/%d/%Y');           # 08/15/2024
say $dt.strftime('%m/%d/%y');           # 08/15/24
say $dt.strftime('%B %d, %Y');          # August 15, 2024

# European formats
say $dt.strftime('%d/%m/%Y');           # 15/08/2024
say $dt.strftime('%d.%m.%Y');           # 15.08.2024
say $dt.strftime('%d-%m-%Y');           # 15-08-2024

# ISO formats
say $dt.strftime('%Y-%m-%d');           # 2024-08-15
say $dt.strftime('%Y-%m-%dT%H:%M:%S');  # 2024-08-15T14:30:45

# Time formats
say $dt.strftime('%H:%M:%S');           # 14:30:45
say $dt.strftime('%I:%M:%S %p');        # 02:30:45 PM
say $dt.strftime('%H:%M');              # 14:30

# Custom formats
say $dt.strftime('%A, %B %d, %Y at %I:%M %p');  # Thursday, August 15, 2024 at 02:30 PM
```

Use strftime with standard format codes for consistent date formatting.  
Consider your audience's cultural expectations when choosing format  
patterns.  

## Error handling and edge cases

Robust error handling for date and time operations.  

```raku
# Handle invalid date input
sub safe-date-creation($year, $month, $day) {
    try {
        return Date.new($year, $month, $day);
        CATCH {
            when X::OutOfRange {
                warn "Date out of range: $year-$month-$day";
                return Nil;
            }
            default {
                warn "Error creating date: {.message}";
                return Nil;
            }
        }
    }
}

my $safe_date = safe-date-creation(2024, 2, 30);  # Returns Nil
say $safe_date // "Invalid date";

# Handle timezone edge cases
sub safe-timezone-conversion($dt, $offset) {
    try {
        return $dt.in-timezone($offset);
        CATCH {
            warn "Invalid timezone offset: $offset";
            return $dt;  # Return original on error
        }
    }
}

# Handle leap year edge cases
sub days-in-month($year, $month) {
    given $month {
        when 1|3|5|7|8|10|12 { 31 }
        when 4|6|9|11 { 30 }
        when 2 { Date.new($year, 1, 1).is-leap-year ?? 29 !! 28 }
        default { die "Invalid month: $month" }
    }
}

say "Days in February 2024: {days-in-month(2024, 2)}";
say "Days in February 2023: {days-in-month(2023, 2)}";
```

Always handle potential exceptions when working with user input or  
external data. Provide meaningful error messages and fallback behavior  
for robust applications.  

## Best practices

Guidelines for effective DateTime usage in Raku applications.  

```raku
# Use appropriate precision for your needs
my Date $event_date = Date.today;           # Events, birthdays, deadlines
my DateTime $log_entry = DateTime.now;      # Logging, audit trails
my Instant $benchmark = Instant.now;        # Performance measurements

# Store UTC, display local
my $utc_timestamp = DateTime.now(:utc);
# ... store $utc_timestamp in database ...
# ... later, convert for display ...
my $local_display = $utc_timestamp.in-timezone($*TZ);

# Validate input early
sub process-date-input($date_string) {
    my $date = try { Date.new($date_string) };
    return Nil without $date;
    
    # Additional business logic validation
    return Nil if $date < Date.new(1900, 1, 1);
    return Nil if $date > Date.today + 365;
    
    return $date;
}

# Use constants for common offsets
constant UTC = 0;
constant EST = -18000;  # -5 hours
constant PST = -28800;  # -8 hours

# Document timezone assumptions
#| Process user registration (assumes UTC input)
sub register-user($name, DateTime $signup_time) {
    # Convert to local time for logging
    my $local_time = $signup_time.in-timezone($*TZ);
    say "User $name registered at $local_time local time";
}
```

Use appropriate temporal types for your data, always store timestamps  
in UTC, validate inputs early, use constants for timezone offsets, and  
document timezone assumptions in your code.

## Daylight saving time

Handling daylight saving time transitions and edge cases.  

```raku
# DST transitions can cause ambiguous times
my $spring_forward = DateTime.new(
    year => 2024, month => 3, day => 10,
    hour => 2, minute => 30,
    timezone => -18000  # EST before DST
);

say "Before DST: $spring_forward";

# Convert to different timezone aware of DST
my $utc_equiv = $spring_forward.utc;
say "UTC equivalent: $utc_equiv";

# DST-aware calculations require careful handling
my $dst_start = Date.new(2024, 3, 10);  # Example DST start
my $dst_end = Date.new(2024, 11, 3);    # Example DST end

sub is-dst-period($date) {
    my $dst_start = Date.new($date.year, 3, 10);
    my $dst_end = Date.new($date.year, 11, 3);
    return $dst_start <= $date <= $dst_end;
}

say "July 15 is DST: {is-dst-period(Date.new(2024, 7, 15))}";
say "January 15 is DST: {is-dst-period(Date.new(2024, 1, 15))}";
```

Daylight saving time creates complexities in timezone handling. Always  
use UTC for storage and be explicit about timezone conversions.  
Consider using timezone libraries for complex DST calculations.  

## Date intervals and scheduling

Working with recurring dates and scheduling patterns.  

```raku
# Weekly intervals
my $start_date = Date.new(2024, 1, 1);
my @weekly_meetings = gather for 0..11 {
    take $start_date + $_ * 7;
};
say "Weekly meetings: @weekly_meetings";

# Monthly intervals (same day each month)
my @monthly_reports = gather for 1..12 {
    take Date.new(2024, $_, 15);
};
say "Monthly reports: @monthly_reports";

# Quarterly intervals
my @quarterly_reviews = gather for 1, 4, 7, 10 {
    take Date.new(2024, $_, 1);
};
say "Quarterly reviews: @quarterly_reviews";

# Last day of each month
my @month_ends = gather for 1..12 {
    my $date = Date.new(2024, $_, 1);
    take Date.new(2024, $_, $date.days-in-month);
};
say "Month ends: @month_ends";
```

Use loops and arithmetic to generate recurring date patterns. Handle  
month-end dates carefully as months have different lengths. Consider  
business rules for holiday adjustments.  

## Working with fiscal years

Calculations based on fiscal year rather than calendar year.  

```raku
# Fiscal year starting July 1
sub fiscal-year($date) {
    return $date.month >= 7 ?? $date.year + 1 !! $date.year;
}

sub fiscal-year-start($fiscal_year) {
    return Date.new($fiscal_year - 1, 7, 1);
}

sub fiscal-year-end($fiscal_year) {
    return Date.new($fiscal_year, 6, 30);
}

my $today = Date.today;
my $fy = fiscal-year($today);
my $fy_start = fiscal-year-start($fy);
my $fy_end = fiscal-year-end($fy);

say "Today: $today";
say "Fiscal Year: $fy";
say "FY Start: $fy_start";
say "FY End: $fy_end";

# Fiscal quarter calculation
sub fiscal-quarter($date) {
    my $fy_start = fiscal-year-start(fiscal-year($date));
    my $months_elapsed = ($date.year - $fy_start.year) * 12 + 
                        ($date.month - $fy_start.month);
    return ($months_elapsed div 3) + 1;
}

say "Current fiscal quarter: {fiscal-quarter($today)}";
```

Fiscal years often differ from calendar years. Create utility functions  
to convert between fiscal and calendar calculations. Handle edge cases  
around fiscal year boundaries.  

## Holiday calculations

Determining holidays and business calendar operations.  

```raku
# Fixed holidays
sub is-new-years($date) { $date.month == 1 && $date.day == 1 }
sub is-christmas($date) { $date.month == 12 && $date.day == 25 }
sub is-independence-day($date) { $date.month == 7 && $date.day == 4 }

# Floating holidays (nth weekday of month)
sub memorial-day($year) {
    # Last Monday in May
    my $may_31 = Date.new($year, 5, 31);
    my $last_monday = $may_31 - (($may_31.day-of-week - 1) % 7);
    return $last_monday;
}

sub labor-day($year) {
    # First Monday in September
    my $sept_1 = Date.new($year, 9, 1);
    my $first_monday = $sept_1 + ((8 - $sept_1.day-of-week) % 7);
    return $first_monday;
}

sub thanksgiving($year) {
    # Fourth Thursday in November
    my $nov_1 = Date.new($year, 11, 1);
    my $first_thursday = $nov_1 + ((11 - $nov_1.day-of-week) % 7);
    return $first_thursday + 21;  # Add 3 weeks
}

my $year = Date.today.year;
say "Memorial Day $year: {memorial-day($year)}";
say "Labor Day $year: {labor-day($year)}";
say "Thanksgiving $year: {thanksgiving($year)}";

# Check if date is a holiday
sub is-holiday($date) {
    return is-new-years($date) || is-christmas($date) || 
           is-independence-day($date) || 
           $date == memorial-day($date.year) ||
           $date == labor-day($date.year) ||
           $date == thanksgiving($date.year);
}

say "Today is holiday: {is-holiday(Date.today)}";
```

Holiday calculations combine fixed dates with floating holidays based  
on weekday rules. Create lookup functions for different holiday  
systems (US, European, etc.) as needed.  

## Age calculations and anniversaries

Precise age calculations and anniversary handling.  

```raku
my $birth_date = Date.new(1990, 5, 15);
my $today = Date.today;

# Precise age calculation
sub calculate-age($birth_date, $reference_date = Date.today) {
    my $age = $reference_date.year - $birth_date.year;
    
    # Adjust if birthday hasn't occurred this year
    if $reference_date.month < $birth_date.month ||
       ($reference_date.month == $birth_date.month && 
        $reference_date.day < $birth_date.day) {
        $age--;
    }
    
    return $age;
}

say "Current age: {calculate-age($birth_date)}";

# Next birthday
sub next-birthday($birth_date, $reference_date = Date.today) {
    my $this_year_birthday = Date.new(
        $reference_date.year,
        $birth_date.month,
        $birth_date.day
    );
    
    if $this_year_birthday >= $reference_date {
        return $this_year_birthday;
    } else {
        return Date.new(
            $reference_date.year + 1,
            $birth_date.month,
            $birth_date.day
        );
    }
}

my $next_bday = next-birthday($birth_date);
my $days_until = $next_bday - $today;
say "Next birthday: $next_bday ({$days_until} days)";

# Anniversary calculations
my $wedding_date = Date.new(2015, 8, 22);
my $years_married = calculate-age($wedding_date);
say "Years married: $years_married";
```

Age calculations must account for leap years and partial years.  
Anniversary functions can reuse age calculation logic with different  
reference dates.  

## Time zone database integration

Working with named time zones and regional considerations.  

```raku
# Time zone offset helpers (simplified examples)
my %timezone_offsets = 
    'EST' => -18000,   # Eastern Standard Time
    'PST' => -28800,   # Pacific Standard Time  
    'GMT' => 0,        # Greenwich Mean Time
    'JST' => 32400,    # Japan Standard Time
    'CET' => 3600;     # Central European Time

sub get-timezone-offset($tz_name) {
    return %timezone_offsets{$tz_name} // 0;
}

# Convert between named timezones
sub convert-timezone($datetime, $from_tz, $to_tz) {
    my $from_offset = get-timezone-offset($from_tz);
    my $to_offset = get-timezone-offset($to_tz);
    
    # Convert to UTC first, then to target timezone
    my $utc = $datetime.clone(timezone => $from_offset).utc;
    return $utc.in-timezone($to_offset);
}

my $meeting_time = DateTime.new(
    year => 2024, month => 8, day => 15,
    hour => 14, minute => 30
);

say "Meeting in EST: {$meeting_time.in-timezone(get-timezone-offset('EST'))}";
say "Meeting in PST: {$meeting_time.in-timezone(get-timezone-offset('PST'))}";
say "Meeting in JST: {$meeting_time.in-timezone(get-timezone-offset('JST'))}";

# Business hours check
sub is-business-hours($datetime, $timezone = 'EST') {
    my $local_time = $datetime.in-timezone(get-timezone-offset($timezone));
    my $hour = $local_time.hour;
    my $dow = $local_time.day-of-week;
    
    return $dow <= 5 && 9 <= $hour <= 17;  # Mon-Fri, 9 AM to 5 PM
}

say "Is business hours: {is-business-hours(DateTime.now)}";
```

Time zone handling becomes complex with DST and regional rules. For  
production applications, consider using dedicated timezone libraries  
that handle historical changes and DST transitions.  

## Date serialization and storage

Converting dates for databases and APIs.  

```raku
# ISO 8601 serialization (recommended for APIs)
my $dt = DateTime.now;
my $iso_string = $dt.iso8601;
say "ISO 8601: $iso_string";

# Parse back from ISO
my $parsed_dt = DateTime.new($iso_string);
say "Parsed: $parsed_dt";

# Unix timestamp serialization
my $timestamp = $dt.posix;
say "Unix timestamp: $timestamp";
my $from_timestamp = DateTime.from-posix($timestamp);
say "From timestamp: $from_timestamp";

# Date-only serialization
my $date = Date.today;
my $date_string = $date.Str;  # YYYY-MM-DD format
say "Date string: $date_string";
my $parsed_date = Date.new($date_string);
say "Parsed date: $parsed_date";

# Custom serialization format
sub serialize-datetime($datetime) {
    return sprintf '%04d%02d%02d%02d%02d%02d',
        $datetime.year, $datetime.month, $datetime.day,
        $datetime.hour, $datetime.minute, $datetime.second;
}

sub deserialize-datetime($compact_string) {
    my $year = $compact_string.substr(0, 4).Int;
    my $month = $compact_string.substr(4, 2).Int;
    my $day = $compact_string.substr(6, 2).Int;
    my $hour = $compact_string.substr(8, 2).Int;
    my $minute = $compact_string.substr(10, 2).Int;
    my $second = $compact_string.substr(12, 2).Int;
    
    return DateTime.new($year, $month, $day, $hour, $minute, $second);
}

my $compact = serialize-datetime($dt);
say "Compact format: $compact";
say "Deserialized: {deserialize-datetime($compact)}";
```

Choose serialization formats based on your needs: ISO 8601 for  
interoperability, Unix timestamps for compactness, or custom formats  
for specific requirements. Always include timezone information.  

## Temporal arithmetic edge cases

Handling complex date arithmetic scenarios.  

```raku
# Month arithmetic with different month lengths
my $jan_31 = Date.new(2024, 1, 31);
say "January 31: $jan_31";

# Adding months to end-of-month dates
sub add-months($date, $months) {
    my $target_year = $date.year;
    my $target_month = $date.month + $months;
    
    # Handle year overflow
    while $target_month > 12 {
        $target_year++;
        $target_month -= 12;
    }
    while $target_month < 1 {
        $target_year--;
        $target_month += 12;
    }
    
    # Handle day overflow (e.g., Jan 31 + 1 month = Feb 28/29)
    my $max_day = Date.new($target_year, $target_month, 1).days-in-month;
    my $target_day = $date.day min $max_day;
    
    return Date.new($target_year, $target_month, $target_day);
}

say "Jan 31 + 1 month: {add-months($jan_31, 1)}";
say "Jan 31 + 2 months: {add-months($jan_31, 2)}";

# Leap year considerations
my $leap_day = Date.new(2024, 2, 29);
say "Leap day + 1 year: {add-months($leap_day, 12)}";

# Business day calculations with holidays
sub add-business-days-no-holidays($date, $days) {
    my $result = $date;
    my $added = 0;
    
    while $added < $days {
        $result++;
        # Skip weekends
        next if $result.day-of-week > 5;
        # Skip holidays (simplified check)
        next if is-holiday($result);
        $added++;
    }
    
    return $result;
}

my $start = Date.new(2024, 12, 20);
say "10 business days from Dec 20: {add-business-days-no-holidays($start, 10)}";
```

Complex date arithmetic requires careful handling of month boundaries,  
leap years, and business rules. Create robust helper functions that  
handle edge cases explicitly.  

## Performance optimization techniques

Optimizing date and time operations for high-performance applications.  

```raku
# Date caching for repeated calculations
my %date_cache;
sub cached-date-creation($year, $month, $day) {
    my $key = "$year-$month-$day";
    return %date_cache{$key} //= Date.new($year, $month, $day);
}

# Batch date operations
sub batch-date-range($start, $end) {
    return gather for 0..($end - $start) {
        take $start + $_;
    };
}

# Use integer arithmetic when possible
sub days-between-fast($date1, $date2) {
    # More efficient than full date subtraction for simple cases
    return ($date2 - $date1).Int;
}

# Pre-calculate common values
constant SECONDS_PER_DAY = 86400;
constant DAYS_PER_WEEK = 7;
constant MONTHS_PER_YEAR = 12;

# Efficient business day counting
sub count-business-days($start, $end) {
    my $total_days = $end - $start;
    my $full_weeks = $total_days div DAYS_PER_WEEK;
    my $remaining_days = $total_days % DAYS_PER_WEEK;
    
    # Count business days in full weeks
    my $business_days = $full_weeks * 5;
    
    # Count business days in remaining days
    my $current = $start + $full_weeks * DAYS_PER_WEEK;
    for 0..^$remaining_days {
        $business_days++ if ($current + $_).day-of-week <= 5;
    }
    
    return $business_days;
}

say "Business days in range: {count-business-days(Date.new(2024, 1, 1), Date.new(2024, 1, 31))}";

# Memory-efficient date iteration
sub process-date-range($start, $end, &callback) {
    my $current = $start;
    while $current <= $end {
        callback($current);
        $current++;
    }
}

# Example: count weekends without storing all dates
my $weekend_count = 0;
process-date-range(
    Date.new(2024, 1, 1),
    Date.new(2024, 12, 31),
    -> $date { $weekend_count++ if $date.day-of-week > 5 }
);
say "Weekends in 2024: $weekend_count";
```

For high-performance applications, cache frequently used dates, use  
integer arithmetic when possible, and process dates in batches rather  
than individually. Consider memory usage for large date ranges.  

## Locale-aware formatting

Formatting dates according to different cultural conventions.  

```raku
# Different date format conventions
sub format-date-us($date) {
    return $date.strftime('%m/%d/%Y');
}

sub format-date-european($date) {
    return $date.strftime('%d/%m/%Y');
}

sub format-date-iso($date) {
    return $date.strftime('%Y-%m-%d');
}

sub format-date-verbose($date) {
    return $date.strftime('%A, %B %d, %Y');
}

my $date = Date.new(2024, 3, 15);
say "US format: {format-date-us($date)}";
say "European format: {format-date-european($date)}";
say "ISO format: {format-date-iso($date)}";
say "Verbose format: {format-date-verbose($date)}";

# Time format conventions
sub format-time-12hour($datetime) {
    return $datetime.strftime('%I:%M:%S %p');
}

sub format-time-24hour($datetime) {
    return $datetime.strftime('%H:%M:%S');
}

my $dt = DateTime.new(2024, 3, 15, 14, 30, 45);
say "12-hour format: {format-time-12hour($dt)}";
say "24-hour format: {format-time-24hour($dt)}";

# Week numbering (ISO 8601 vs others)
sub iso-week-number($date) {
    # Simplified ISO week calculation
    my $jan1 = Date.new($date.year, 1, 1);
    my $days_from_jan1 = $date.day-of-year - 1;
    my $week = ($days_from_jan1 + $jan1.day-of-week - 1) div 7 + 1;
    return $week;
}

say "ISO week number: {iso-week-number(Date.new(2024, 3, 15))}";

# Different first day of week conventions
sub week-start-sunday($date) {
    my $dow = $date.day-of-week;
    return $date - ($dow % 7);  # Sunday = 0
}

sub week-start-monday($date) {
    my $dow = $date.day-of-week;
    return $date - ($dow - 1);  # Monday = 1
}

say "Week start (Sunday): {week-start-sunday(Date.new(2024, 3, 15))}";
say "Week start (Monday): {week-start-monday(Date.new(2024, 3, 15))}";
```

Different locales have varying conventions for date formatting, week  
numbering, and first day of week. Create format functions that can  
be easily switched based on user preferences or locale settings.  

## Integration patterns

Common patterns for integrating DateTime with other systems and libraries.  

```raku
# Database timestamp patterns
class DatabaseRecord {
    has DateTime $.created_at;
    has DateTime $.updated_at;
    
    method touch() {
        $!updated_at = DateTime.now(:utc);
    }
    
    method age() {
        return DateTime.now(:utc) - $.created_at;
    }
}

my $record = DatabaseRecord.new(
    created_at => DateTime.now(:utc),
    updated_at => DateTime.now(:utc)
);

say "Record age: {$record.age} seconds";

# API response formatting
sub format-api-response(%data) {
    return %(
        data => %data,
        timestamp => DateTime.now(:utc).iso8601,
        server_time => DateTime.now.Str
    );
}

# Logging with timestamps
sub log-message($level, $message) {
    my $timestamp = DateTime.now.strftime('%Y-%m-%d %H:%M:%S');
    say "[$timestamp] $level: $message";
}

log-message('INFO', 'Application started');

# Caching with expiration
class TimedCache {
    has %.cache;
    has %.expiry;
    
    method set($key, $value, $ttl_seconds = 3600) {
        %.cache{$key} = $value;
        %.expiry{$key} = DateTime.now + Duration.new($ttl_seconds);
    }
    
    method get($key) {
        return Nil unless %.cache{$key}:exists;
        
        if DateTime.now > %.expiry{$key} {
            %.cache{$key}:delete;
            %.expiry{$key}:delete;
            return Nil;
        }
        
        return %.cache{$key};
    }
}

my $cache = TimedCache.new;
$cache.set('user:123', { name => 'Alice' }, 300);  # 5 minute TTL
say "Cached value: {$cache.get('user:123')}";

# Event scheduling
class EventScheduler {
    has @.events;
    
    method schedule($name, $datetime) {
        @.events.push: %( name => $name, when => $datetime );
        @.events = @.events.sort: *<when>;
    }
    
    method next-event() {
        my $now = DateTime.now;
        return @.events.first: *<when> > $now;
    }
    
    method overdue-events() {
        my $now = DateTime.now;
        return @.events.grep: *<when> < $now;
    }
}

my $scheduler = EventScheduler.new;
$scheduler.schedule('Meeting', DateTime.now + Duration.new(hours => 2));
$scheduler.schedule('Deadline', DateTime.now + Duration.new(days => 1));

if my $next = $scheduler.next-event() {
    say "Next event: {$next<name>} at {$next<when>}";
}
```

Integration patterns help create consistent DateTime usage across  
applications. Use UTC for storage, provide clear API contracts, and  
implement proper caching and scheduling mechanisms.  