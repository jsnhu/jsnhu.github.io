---
title:  "Flexible shift scheduling with Excel/JuMP: UBC LIFE Collegium"
published: false
share: false
---

*Thanks to the CAs of LIFE Collegium 2018-19 who made this project possible and for making my first year extremely enjoyable. I will thoroughly miss you all.*

[See my GitHub for the Excel sheet and full Julia code.](https://github.com/jsnhu/life-collegium-schedule)

This scheduling problem is much more exciting than [my first project](https://jsnhu.github.io/spl-scheduling/) because of the nature of flexible shifts. In a predetermined shift problem, the shifts are decided first and then employees provide their availability only for those shifts. However, for this flexible shift problem, employees first give their availability for all relevant working hours. Then, the shifts are created based on employee availability (and on various constraints).

In creating a flexible assignment model, the optimization variable, objective function, and constraints become much more interesting. Since employees now give their availability for time and day instead of simple set shifts, the data can naturally be represented with an extra dimension. The objective function must now reward continuous shifts to avoid scattered, "swiss-cheese" schedules. Finally, there are new constraint considerations such as minimum shift length.

### A Very Quick Overview

Employees provide their availability and preferences for the week within a timetable. Every employee's availability timetable is collected on a common Excel sheet like so:

<img src="/assets/images/life-scheduling/av2.PNG">

The values in the tables denote the following:

<img src="/assets/images/life-scheduling/legend1.PNG">

Now, the assignment problem is solved with Gurobi and JuMP. The finished schedule is produced and read into another sheet within the same workbook.

<img src="/assets/images/life-scheduling/output1.PNG">

Here, the shifts are shown as follows:

<img src="/assets/images/life-scheduling/legend2.PNG">

The schedule takes into account:
* Minimum shift length
* Maximum number of opening/closing shifts
* Maximum/minimum weekly working hours
* Maximum/minimum number of employees working a given timeslot
* A collective meeting time in which all employees are scheduled.

### Problem Specifications
*The problem is described with respect to the needs of UBC LIFE Collegium 2018-19, but serves as a basic framework for flexible shift scheduling.*

The LIFE Collegium is open from Monday to Friday, 08:00 to 19:00. We break each day into 22 half-hour slots. For each day, the first shift (08:00) is the opening shift and the last or 22nd shift (18:30) is the closing shift.

There are eight "Collegia Advisors" (CAs) who are employed to supervise and work at LIFE. One CA is designated as the "Senior CA" and has special scheduling considerations different from the other seven ("CA" will henceforth refer to the seven "regular" CAs and the "Senior CA" will explicitly be referred to as such).

#### Regular CAs

Each CA provides a timetable of their availability and preferences using the following numerical system:

<img src="/assets/images/life-scheduling/legend1.png">

Each provides a 22 x 5 table like so:

<img src="/assets/images/life-scheduling/av3.png">

(Note: CAs are UBC students so blocks of unavailability are often classes.)

We collect all of the employee availability timetables into one Excel sheet.

<img src="/assets/images/life-scheduling/av2.png">

#### The Senior CA

Our scheduling methodology is to accommodate the seven CAs as best as possible and then schedule the Senior CA to pick up any leftover shifts. Hence, our output will only schedule the Senior CA when absolutely necessary. To do this, the Senior CA's timetable will appear as such:

<img src="/assets/images/life-scheduling/seniorca.png">

Excluding unavailable times, (-5) is the blanket availability score. Essentially, the Senior CA should only be assigned to slots that are highly negative (-99 or -100) for the other CAs and not when other CAs are available (with a score greater than or equal to 0).

The model does not force a minimum number of working hours for the Senior CA. The idea is that the Senior CA should assign themselves more slots as they see fit after seeing their "mandated" slots.

#### The Placeholder

It is possible that none of the CAs or the Senior CA can work a certain time slot (which occurred in 2018-19 Term 2). In this case, a "substitute" or "placeholder CA" is required. This placeholder (called "Xx") has the following timetable:

<img src="/assets/images/life-scheduling/xx.png">

The blanket (-50) availability score ensures that they are only assigned when all other staff members have scores of (-100) or (-99). The final output will show which hours, if any, a substitute must be hired.

### Reading into Julia with Taro
In our Excel sheet, Xx is always the first timetable, followed by the Senior CA, then the rest of the CAs in arbitrary order.

The number of employees must be manually entered into cell B27 so as to tell the program how far to read into the sheet. B28, which includes the placeholder, is ultimately read by the program and becomes `staff`.

<img src="/assets/images/life-scheduling/details.png">

Now, we use the [Taro](https://github.com/aviks/Taro.jl) package to read the tables in Excel into Julia arrays. We take advantage of the even spacing of the 22 x 5 tables to obtain the amount specified in B28.

```julia
# get staff availability tables
# top left cell on row 2, bot right cell on row 23
# each table separated by 7 cells

staff_array = []                        # with preference/availability data
staff_dict  = Dict{Integer, String}()   # with names of staff

for i in 0:staff - 1
    range = string(numtocol(7 * i + 2),
                    "2:",
                    numtocol(7 * i + 6),
                    "23")
    push!(staff_array,
        DataFrame(Taro.readxl("availability.xlsx", "availability",
        range, header = false)))
    staff_dict[i + 1] = String(getCellValue(getCell(getRow(getSheet(
        Workbook("availability.xlsx"), "availability"), 0), 7 * i)))
end
```

Unlike the 2-dimensional availability array in the SPL scheduling project, this availability array is 3-dimensional (22 slots/day x 5 days x 9 employees):

```julia
# create 3D availability array
av_matrix = Array{Int8}(undef, 22, 5, staff)

for k in 1:staff
    for i in 1:22
        for j in 1:5
            av_matrix[i,j,k] = Matrix(staff_array[k])[i,j]
        end
    end
end
```
### Model

#### Variable
We use Gurobi for our model. Our binary assignment variable is defined as follows:

```julia
# 22 x 5 x staff binary assignment 3d matrix
# 1 if employee k assigned to shift (i,j), 0 otherwise
@variable(m, x[1:22, 1:5, 1:staff], Bin)
```
#### Objective Function
#### Constraints
#### Result
#### Extensions
### Improvements
