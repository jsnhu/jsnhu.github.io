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
