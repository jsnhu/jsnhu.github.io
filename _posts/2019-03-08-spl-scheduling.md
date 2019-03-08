---
title:  "Library shift scheduling with Excel/JuMP"
published: true
share: false
---

*This was originally a final project in Linear Algebra at Quest University completed with my classmates, Casey Lake and Edward Zuo, under Dr. Richard Hoshino. The data was obtained in August, 2017. I have since migrated it from Maple to an Excel/JuMP system for ease of use and updatability.*

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

We create an 8x17 preference table in Excel where the entry in the *ith* row, *jth* column represents employee *i*'s preference for shift *j*. The entries are defined as:
* **C:** Cannot work this shift.
* **U:** Unpreferred shift.
* **N:** Neutral preference.
* **P:** Preferred shift.

Automatically, **C** is the assumed preference to desk shifts for shelf employees and shelf shifts for desk employees.

![](images/preftable.png "Sample of the preference table.")

We translate these preferences into numerical scores to be used in the model's objective function.
* **C** = -1000
* **U** = -1
* **N** = 0
* **P** = 1

![](images/preftable1.png "Sample of the preference table with numerical values.")

Now, we use the [Taro](https://github.com/aviks/Taro.jl) package to read Excel files into a Julia Dataframe.

```julia
# get staff, shift, preference tables
staff_df = DataFrame(Taro.readxl("table.xlsx", "Dictionary", staff_key))
shift_df = DataFrame(Taro.readxl("table.xlsx", "Dictionary", shift_key))
pref_df  = DataFrame(Taro.readxl("table.xlsx", "PrefMatrix", pref_key, header = false))
```

In Julia, we will classify the employees into arrays according to their type and the shifts into arrays according to their type and day of the week. These arrays will make construction of the model constraints easier.

```julia
# categorize staff by their type
both_staff = Int64[]
desk_staff = Int64[]
shel_staff = Int64[]

for k in 1:size(staff_df,1)
    if staff_df[k,:Type] == "both"
        push!(both_staff,k)
    elseif staff_df[k,:Type] == "desk"
        push!(desk_staff,k)
    else
        push!(shel_staff,k)
    end
end
```

We will use the free solver, GLPKSolver, for our model. Our model will maximize an objective function defined as follows:
```julia
# maximize preference score sum
@objective(m, Max, sum(pref_matrix[i,j]*x[i,j] for i in 1:staff, j in 1:shift))
```
In other words, the solver will produce a schedule which tries to maximizes the number of people who get shifts they prefer.


Next, convert our preference table into a preference matrix and define a binary assignment matrix. This is the variable to be optimized in the model. Note that *staff* represents the total number of employees and *shift* represents the total number of shifts. They are 8 and 17 respectively but can easily be changed to account for variations in the scheduling problem.

```julia
# staff x shift binary assignment matrix
# 1 if employee i assigned to shift j, 0 otherwise
@variable(m, x[1:staff, 1:shift], Bin)
```

Continuing, we can now define the hard constraints of our model. A valid schedule cannot violate these constraints. These can be generally interpreted as sums of rows or columns in our assignment matrix. Code for the first constraint is shown as an example (see GitHub for the rest).
#### Constraints
1. There is exactly one person per shift.
```julia
for j in 1:shift
    @constraint(m, sum( x[i,j] for i in 1:staff) == 1)
end
```
2. Each person works a maximum four shifts per week.
3. Each person works a minimum one shift per week.
4. No employee works both Saturday and Sunday in one weekend.
```julia
for i in 1:staff
    @constraint(m, sum( x[i,j] for j in union(sat_shift,sun_shift)) <= 1)
end
```
5. Desk employees cannot work shelving shifts.
6. Shelving employees cannot work desk shifts.
7. Each person works a maximum one shift per day.

Finally, we run the solver and write the results into a DataFrame.

```julia
Objective value: 1.0
17×3 DataFrame
│ Row │ Employee │ Shift │ Score │
│     │ Int64    │ Int64 │ Int64 │
├─────┼──────────┼───────┼───────┤
│ 1   │ 1        │ 6     │ -1    │
│ 2   │ 1        │ 8     │ -1    │
│ 3   │ 1        │ 14    │ 0     │
│ 4   │ 2        │ 9     │ 0     │
│ 5   │ 2        │ 11    │ 0     │
│ 6   │ 2        │ 13    │ 0     │
│ 7   │ 2        │ 16    │ 0     │
│ 8   │ 3        │ 3     │ 1     │
│ 9   │ 4        │ 1     │ 1     │
│ 10  │ 4        │ 2     │ 1     │
│ 11  │ 4        │ 5     │ 0     │
│ 12  │ 5        │ 4     │ 0     │
│ 13  │ 6        │ 10    │ 0     │
│ 14  │ 6        │ 15    │ 0     │
│ 15  │ 7        │ 12    │ 0     │
│ 16  │ 7        │ 17    │ 0     │
│ 17  │ 8        │ 7     │ 0     │
```

We see that a possible optimal schedule (there could be others with objective value 1.0) only assigns employee 1 to unpreferred shifts. In general, most employees are satisfied considering our original preference matrix had very few **P** entries.

### Future extensions:
* Generally, desk shifts have higher pay than shelving shifts. Thus, those who can work both may want to be guaranteed a desk shift. If we look at the current schedule, employee 2 is relegated to picking up the left-over shelving shifts.
* A full-time employee goes on vacation and leaves five new shifts to be covered or a new part-time employee is hired. This will test how easy it is to update the system to account for changes to the number of staff or shifts.

### Future improvements:
* Write the end result to a Excel.
* Automatically read the boundaries of the preference, staff, and shift tables in Excel to avoid manual inputting of cell ranges.
* Eventually migrate to Google Spreadsheet to use preference updating with Google Forms and easy creation of a Google Calendar.
