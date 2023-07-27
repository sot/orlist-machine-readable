# OR list machine-readable comments

## Requirements draft

The OR list machine-readable comments specification describes the format and content for
embedding machine-readable information into `COMMENT` blocks within the OR list
file. This is to be considered a supplement to the `OBSERVATION REQUESTS` section 3.24
of OP-19, where the `COMMENT` block is defined in 3.24.3.5.

This requirements document is controlled by the FSDS process and changes require FSDS
approval.

`COMMENT` blocks with an `ID` value less than 38000 (hereafter "machine-readable
comments") shall consist entirely of text that conforms to the [YAML data specification
1.2](https://yaml.org/spec/1.2.2/).

Each `OBS` statement shall have one and exactly one associated machine-readable comment.

`COMMENT` blocks with an `ID` value greater than 38000 (hereafter supplementary
comments) shall consist of supplementary information intended only for human reading.

**No machine applications shall parse or use information in the supplementary comments,
with one exception below.** Supplementary comments are not subject to any control
process and may change without notice. A notable example of such supplementary content
is the `Target Summary` table.

- Exception: the `ZERO-OFFSET AIMPOINTS` table embedded between
  `***ZERO-OFFSET AIMPOINTS START` and `***ZERO-OFFSET AIMPOINTS END` lines is
  machine-readable and used by existing software.

As a corollary, any scheduling or constraint information provided in the OR list which
is required for schedule generation shall be contained within either the `OBS` statement
or associated machine-readable comment.

The data fields contained within machine-readable comments shall conform to the
`OR list machine-readable comment` specification.


### OR list machine-readable comment format

A machine-readable comment consists of YAML-formatted text that represents the
following structured data:

- `absolute_mon_window` (map, optional)
  - *Provided if ObsID has monitor or group constraint and preceding ObsID has a
    schedule date.*
  - Keys:
    - `obsid` (int, required)
      - Preceding ObsID in monitor series or group
    - `start` (str, required)
      - Window start (YYYY:DDD:HH:MM:SS.sss)
    - `stop` (str, required)
      - Window stop (YYYY:DDD:HH:MM:SS.sss)
- `acis_fp_limit` (float, required for ACIS)
  - ACIS focal plane limit temperature (degC)
- `chips` (map, required for ACIS)
  - Information on ACIS chip counts
  - Keys:
    - `dropped` (int, required)
      - Number of dropped chips in planned schedule
    - `optional` (int, required)
      - Number of optional chips specified by observer
    - `requested` (int, required)
      - Number of requested chips specified by observer
- `comment` (str, optional)
  - Supplementary comment for humans. No applications shall parse this information
    and the text is subject to change without notice.
- `coordination_window` (map, optional)
  - *Provided if ObsID has coordination constraint.*
  - Keys:
    - `start` (str, required)
      - Window start (YYYY:DDD:HH:MM:SS.sss)
    - `stop` (str, required)
      - Window stop (YYYY:DDD:HH:MM:SS.sss)
- `cycle_number` (int, required)
  - Observation cycle number
- `drop_chip_si_modes` (list of str, required for ACIS)
  - List of six SI modes corresponding to dropping 0, 1, 2, 3, 4, 5 ACIS chips respectively.
  - `'NULL'` indicates that no SI mode is available for that number of
    dropped chips.
  - Single quotes around `NULL` is required.
- `obs_group` (list of int, optional)
  - *Provided if this ObsID has a group constraint.*
  - List of ObsIDs in group
- `obs_group_duration` (float, optional)
  - Duration (days) within which the group ObsIDs must be scheduled.
- `phase_window` (table, optional)
  - Table of allowed start and stop times for phase constraints.
  - Columns:
    - `start` (str)
      - Window start (YYYY:DDD:HH:MM:SS.sss)
    - `stop` (str)
      - Window stop (YYYY:DDD:HH:MM:SS.sss)
- `pointing` (table, optional)
  - *Provided if ObsID has roll-dependent pointing constraint.*
  - Table that describes the roll-dependent pointing constraint.
  - Columns:
    - `roll_start` (float)
      - Roll window start (deg, 0.0 : 360.0)
    - `roll_stop`
      - Roll window stop (deg, 0.0 : 360.0)
      - `roll_start` < `roll_stop`
    - `y_target_offset`, `z_target_offset`, `RA`, `Dec`, `z_sim_offset`.
- `relative_mon_window` (map, optional)
  - *Provided if ObsID has monitor or group constraint.*
  - `obsid` (int, required)
    - Preceding ObsID in monitor series or group for relative monitor window constraint.
  - `start` (str, required)
    - Window start (DDD:HH:MM:SS.sss) relative to reference ObsID stop ?? (decimal days??).
  - `stop` (str, required)
    - Window stop (DDD:HH:MM:SS.sss) relative to reference ObsID stop ??.
- `roll` (table, optional)
  - *Provided if ObsID has a roll constraint.*
  - Table that provides allowed roll windows.
  - *Need a convention for wrapping*
  - Columns: `min`, `max`
- `sequence_number` (int, required)
  - Observation sequence number
- `slosh` (int, float?, optional)
  - *Provided if sequence has unscheduled time.*
  - Amount of time (ks) available to slosh in / out. ??
- `split_duration` (float, optional)
  - *Provided if ObsID has a split interval constraint.*
  - Number of days within which all splits must be completed.

### Intervals

Intervals of time or other quantities such as roll angle shall be represented
with a `start` and `stop` value. The interval is half open, meaning `start <= value < stop`.

### Roll angles and intervals

`roll` refers to spacecraft attitude roll angle i.e. `(RA, dec, roll)`. This shall be represented as a float in the range `0.0 <= roll <= 360.0`.

Roll intervals shall be represented with a start and stop roll value where `roll_start <= roll < roll_stop`. In the case of a roll interval that covers 0.0 deg, this would be broken into two intervals. For instance `330.15 : 360.0` and `0.0 : 20.51`.

## Questions

### Coordination window

What is the intent of the `coordination_window` field?  One example includes a `freeform_constraint` that implies this may be associated with a timing constraint:
```
  Window Constraint requirements exist for observation.
    WINDOW=(2023:152:00:00:00,2023:212:00:00:00)
```

### Monitor windows?

Can there be multiple `relative_mon_window` or `absolute_mon_window`?

### Table format

Should the embedded tables be human- and machine-readable or only machine-readable?
Current format is a hybrid with space around comma but not really human-readable.
```
#roll_start, roll_stop, y_target_offset, z_target_offset, RA, Dec, z_sim_offset
   0.0, 20.0, 0.008333, 0.0, 277.292632, -12.875559, 0.0
  20.0, 42.0, 0.0, -0.016667, 277.279615, -12.857489, 0.0
```
For  human-readable a simple format would be similar to other OR list tables. The
line of dashes unambiguously defines the column boundaries and this is an easy format
to read and write, but it is wider.
```
roll_start roll_stop y_target_offset z_target_offset         RA        Dec z_sim_offset
---------- --------- --------------- --------------- ---------- ---------- ------------
       0.0      20.0        0.008333             0.0 277.292632 -12.875559          0.0
      20.0      42.0             0.0       -0.016667 277.279615 -12.857489          0.0
```
For machine-readable we should just standard comma-separated values with no embedded
spaces:
```
# roll_start,roll_stop,y_target_offset,z_target_offset,RA,Dec,z_sim_offset
0.0,20.0,0.008333,0.0,277.292632,-12.875559,0.0
20.0,42.0,0.0,-0.016667,277.279615,-12.857489,0.0
```

### Table column names

Should we adopt a convention that all column names are lower-case? Currently the names
are a mix of mostly lower case, some capitalized, and some upper-case.

## Implementation details

### Multiline literal strings

YAML supports including a multiline literal string using the `|` character, see the
[`ASCII art` example](https://yaml.org/spec/1.2.2/#23-scalars). In the context of the
YAML ORlist there can be two additional characters after the `|`. For example:
```
star_field_constraints: !table |2-
   #ROLL, nominal, creep
   81.00, -11.10, -10.10
   ...
  100.00, -11.10, -10.80
  101.00, -11.20, -10.80
```
- The `2` indicates that the indentation of this block is 2 characters. By default the
  indentation is defined by the indentation of the first line. In this case the first
  line has 3 blank spaces but later in the block there are only 2.
 - The `-` indicates that the final newline of the text block is suppressed.

 ### `!table` tag

In the example above, the `!table` element is a YAML tag that explicitly identifies that
element as a table. A YAML parser expects to have a custom parser registered that
provides special handling for the element text.

### Allowed depth

YAML supports arbitrary nested structures. The current spec allows each element to be
either a scalar, table, list of scalars, or dict of scalars. Should we formalize that
restriction?

### Perl parser

I have written a rough prototype of a machine-readable comment reader for Perl using
`YAML::XS`. Any interest in that?
