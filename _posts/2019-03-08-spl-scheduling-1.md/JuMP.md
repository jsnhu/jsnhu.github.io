---
title:  "Draft:!!! Library shift scheduling with Excel/JuMP"
published: true
share: false
---

*This was originally a final project in Linear Algebra at Quest University completed with my classmates Casey Lake and Edward Zuo under Dr. Richard Hoshino. The data was obtained in August, 2017. I have since migrated it from Maple to an Excel/JuMP system for ease of use and updatability.*

## Problem Specifications

The Squamish Public Library has eight part-time employees who must cover seventeen different shifts throughout the week. These library shifts are classified into two types: shelving shifts and circulation (desk) shifts. Certain employees can only work certain shifts. We refer to these employees by their initials:
* Three employees can *only* work desk shifts: KP, JP, EV.
* Three employees can *only* work shelving shifts: DB, WA, TF.
* Two employees can work *either* desk or shelving shifts: VG, ND.

We assign labels to the employees as follows:

| i | Name | Type  |
|---|------|-------|
| 1 | VG   | both  |
| 2 | ND   | both  |
| 3 | KP   | desk  |
| 4 | JP   | desk  |
| 5 | EV   | desk  |
| 6 | DB   | shelf |
| 7 | WA   | shelf |
| 8 | TF   | shelf |

We assign labels to the seventeen shifts in the week:

| j  | Day | Time        | Type  |
|----|-----|-------------|-------|
| 1  | Mon | 10:00-14:00 | desk  |
| 2  | Thu | 13:30-17:30 | desk  |
| 3  | Fri | 13:30-17:30 | desk  |
| 4  | Sat | 12:00-16:00 | desk  |
| 5  | Sun | 12:00-16:00 | desk  |
| 6  | Mon | 10:00-14:00 | shelf |
| 7  | Mon | 16:30-20:30 | shelf |
| 8  | Tue | 10:00-14:00 | shelf |
| 9  | Tue | 16:30-20:30 | shelf |
| 10 | Wed | 10:00-14:00 | shelf |
| 11 | Wed | 16:30-20:30 | shelf |
| 12 | Thu | 10:00-14:00 | shelf |
| 13 | Thu | 16:30-20:30 | shelf |
| 14 | Fri | 10:00-16:30 | shelf |
| 15 | Sat | 10:00-16:30 | shelf |
| 16 | Sun | 10:00-14:00 | shelf |
| 17 | Sun | 10:00-16:30 | shelf |

We create a preference matrix where the entry in the *ith* row, *jth* column represents employee *i*'s preference for shift *j*. The entries are defined as:
* -1000: Cannot work this shift.
* -1: Unpreferred shift.
* 0: Neutral preference.
* 1: Preferred shift.
