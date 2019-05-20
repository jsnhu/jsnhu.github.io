---
title:  "Flexible shift scheduling with Excel/JuMP: UBC LIFE Collegium"
published: false
share: false
---

*Thanks to the CAs of LIFE Collegium 2018-19 who made this project possible and for making my first year extremely enjoyable. I will thoroughly miss you all.*

[See my GitHub for the Excel sheet and full Julia code.](https://github.com/jsnhu/life-collegium-schedule)

This scheduling problem is significantly more exciting than [my first project](https://jsnhu.github.io/spl-scheduling/) because of the nature of flexible shifts. In a predetermined shift problem, the shifts are decided first and then employees provide their availability only for those shifts. However, for this flexible shift problem, employees first give their availability for all relevant working hours. Then, the shifts are created based on employee availability (and on various constraints).

In creating a flexible assignment model, the optimization variable, objective function, and constraints become much more interesting. Since employees now give their availability for time and day instead of simple set shifts, the data can naturally be represented with another dimension. The objective function must now reward continuous shifts to avoid scattered, illogical schedules (e.g. 1 hr shift, 1 hr break, 1 hr shift, 1 hr break, 1 hr shift). Finally, constraints must consider minimum shift length and number of opening/closing shifts.

### A Very Quick Overview

Employees provide their availability and preferences for the week within a timetable. Every employee's availability timetable is collected on a common Excel sheet like so:

<img src="/assets/images/life-scheduling/av2.PNG">

The values in the tables denote the following:

<img src="/assets/images/life-scheduling/legend1.PNG">

Now, the assignment problem is solved with Gurobi and JuMP. The finished schedule is produced and read into another sheet within the same workbook.

<img src="/assets/images/life-scheduling/output1.PNG">

Here, the shifts are shown as follows:

<img src="/assets/images/life-scheduling/legend2.PNG">
### Problem Specifications

<img src="/assets/images/life-scheduling/">


Now, we use the [Taro](https://github.com/aviks/Taro.jl) package to read Excel sheets into Julia DataFrames. We read the staff, shift, and preference score tables.

### Objective Function
### Constraints
### Result
### Extensions
### Improvements
