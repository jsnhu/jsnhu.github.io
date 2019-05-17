---
title:  "Predetermined shift scheduling with Excel/JuMP: Squamish Public Library"
published: true
share: false
---

*This was originally a final project in Linear Algebra at Quest University completed with my classmates, Casey Lake and Edward Zuo, under Dr. Richard Hoshino. The data was obtained in August, 2017. I have since migrated it from Maple to an Excel/JuMP system for ease of use and updatability.*

[See my GitHub for the Excel sheet and full Julia code.](https://github.com/jsnhu/spl-schedule)

### Problem Specifications

The Squamish Public Library has eight part-time employees who must cover seventeen different shifts throughout the week. These library shifts are classified into two types: shelving shifts and circulation (desk) shifts. Certain employees can only work certain shifts. We refer to these employees by their initials:
* Three employees can *only* work desk shifts: KP, JP, EV.
* Three employees can *only* work shelving shifts: DB, WA, TF.
* Two employees can work *either* desk or shelving shifts: VG, ND.

We have an Excel sheet which contains all of the data about employees and shifts called "Dictionary" where we enumerate the employees with a value *i* and shifts with a value *j*.

<img src="/assets/images/spl-scheduling/dictionarysheet.png">

Each employee gives us their availability and preferences. We create an 8x17 preference table in Excel where the entry in the *ith* row, *jth* column represents employee *i*'s preference for shift *j*. The entries are defined as:
* **P:** Preferred shift.
* **N:** Neutral preference.
* **U:** Unpreferred shift.
* **C:** Cannot work this shift (unavailable).

Automatically, **C** is the assumed preference with desk shifts for shelf-exclusive employees and shelf shifts for desk-exclusive employees. Here is a sample of our preference table.

<img src="/assets/images/spl-scheduling/newpref.PNG">

We translate these preferences into numerical scores to be used in the model's objective function.
* **P** = 1
* **N** = 0
* **U** = -1
* **C** = -1000

In general, these values are arbitrary. For now, the important point is that preferred shifts are worth more than neutral shifts which are worth more than unpreferred shifts. Unavailable shifts are very negative as to immediately alert us when the schedule is invalid by inspection of the objective score (defined later). In the future, we may discuss possible uses of changing the relative weight of these scores.

In another sheet, we have the translated score values. These are the values which will be read into Julia.

<img src="/assets/images/spl-scheduling/newpref1.PNG">

Now, we use the [Taro](https://github.com/aviks/Taro.jl) package to read Excel sheets into Julia DataFrames. We read the staff, shift, and preference score tables.

```julia
# get staff, shift, preference tables
staff_df = DataFrame(Taro.readxl("table.xlsx", "Dictionary", staff_key))
shift_df = DataFrame(Taro.readxl("table.xlsx", "Dictionary", shift_key))
pref_df  = DataFrame(Taro.readxl("table.xlsx", "PrefMatrix", pref_key, header = false))
```

In Julia, we will classify the employees into arrays according to their type (desk/shelf/both). Similarly, we classify the shifts into arrays according to their type and also day of the week. These arrays will make construction of the model constraints easier later on.

Below is an example of employee classification.
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
### Objective Function
We will use GLPKSolver for our optimization. Our model will maximize an objective function defined as follows:
```julia
# maximize preference score sum
@objective(m, Max, sum(pref_matrix[i,j]*x[i,j] for i in 1:staff, j in 1:shift))
```
In other words, the solver will produce a schedule which maximizes the sum of the preference scores of assigned employees (called the objective score).
* Assigning an employee to a preferred shift adds to the objective score (+1).
* Assigning an employee to a neutral shift does not change the objective score (0).
* Assigning an employee to an unpreferred shift subtracts from the objective score (-1).
* If an employee is assigned to an unavailable shift, the schedule is considered invalid (-1000).

We want as many people as possible to get preferred shifts in order to have a higher objective score.

Next, we convert our preference score table into a preference score matrix and we define a binary assignment matrix. This is the variable to be optimized in the model. Note that `staff` represents the total number of employees and `shift` represents the total number of shifts. They are 8 and 17 respectively but automatically update when the number of employees/shifts change in the Excel sheet.

```julia
# staff x shift binary assignment matrix
# 1 if employee i assigned to shift j, 0 otherwise
@variable(m, x[1:staff, 1:shift], Bin)
```

Continuing, we can now define the hard constraints of our model. A valid schedule cannot violate these constraints. Code for the first and fourth constraint are shown as examples (see GitHub for the rest).
### Constraints
1. There is exactly one person per shift.
```julia
for j in 1:shift
    @constraint(m, sum( x[i,j] for i in 1:staff) == 1)
end
# in the assignment matrix, the sum of each column should be exactly 1
# since each column represents a shift and those assigned to it
```
2. Each person works a maximum four shifts per week.
3. Each person works a minimum one shift per week.
4. No employee works both Saturday and Sunday in one weekend.
```julia
for i in 1:staff
    @constraint(m, sum( x[i,j] for j in union(sat_shift,sun_shift)) <= 1)
end
# here we use the category arrays from earlier to construct this constraint easily
# rather than manually enumerating the j-value of sat/sun shifts
```
5. Desk employees cannot work shelving shifts.
6. Shelving employees cannot work desk shifts.
7. Each person works a maximum one shift per day.

### Result

Finally, we run the solver and write the results into a DataFrame.

```
Objective value: 1.0
17×7 DataFrame
│ Row │ Employee │ Name   │ Shift │ Day    │ Time        │ Type   │ Score │
│     │ Int64    │ String │ Int64 │ String │ String      │ String │ Int64 │
├─────┼──────────┼────────┼───────┼────────┼─────────────┼────────┼───────┤
│ 1   │ 1        │ VG     │ 6     │ Mon    │ 10:00-14:00 │ shelf  │ -1    │
│ 2   │ 1        │ VG     │ 8     │ Tue    │ 10:00-14:00 │ shelf  │ -1    │
│ 3   │ 1        │ VG     │ 14    │ Fri    │ 10:00-16:30 │ shelf  │ 0     │
│ 4   │ 2        │ ND     │ 9     │ Tue    │ 16:30-20:30 │ shelf  │ 0     │
│ 5   │ 2        │ ND     │ 11    │ Wed    │ 16:30-20:30 │ shelf  │ 0     │
│ 6   │ 2        │ ND     │ 13    │ Thu    │ 16:30-20:30 │ shelf  │ 0     │
│ 7   │ 2        │ ND     │ 16    │ Sun    │ 10:00-14:00 │ shelf  │ 0     │
│ 8   │ 3        │ KP     │ 3     │ Fri    │ 13:30-17:30 │ desk   │ 1     │
│ 9   │ 4        │ JP     │ 1     │ Mon    │ 10:00-14:00 │ desk   │ 1     │
│ 10  │ 4        │ JP     │ 2     │ Thu    │ 13:30-17:30 │ desk   │ 1     │
│ 11  │ 4        │ JP     │ 5     │ Sun    │ 12:00-16:00 │ desk   │ 0     │
│ 12  │ 5        │ EV     │ 4     │ Sat    │ 12:00-16:00 │ desk   │ 0     │
│ 13  │ 6        │ DB     │ 10    │ Wed    │ 10:00-14:00 │ shelf  │ 0     │
│ 14  │ 6        │ DB     │ 15    │ Sat    │ 10:00-16:30 │ shelf  │ 0     │
│ 15  │ 7        │ WA     │ 12    │ Thu    │ 10:00-14:00 │ shelf  │ 0     │
│ 16  │ 7        │ WA     │ 17    │ Sun    │ 10:00-16:30 │ shelf  │ 0     │
│ 17  │ 8        │ TF     │ 7     │ Mon    │ 16:30-20:30 │ shelf  │ 0     │
```

We see that a possible optimal schedule (there could be others with objective value 1.0, but certainly none greater) assigns VG to unpreferred shifts. However, most employees are satisfied in general considering our original preference matrix had very few **P** entries.

Taro also allows us to write the assignment matrix into an Excel workbook (Note: the assignment matrix is written into a different workbook than the preferences. Anecdotally, writing into an Excel file may wipe or corrupt the file.)

<img src="/assets/images/spl-scheduling/assnmatrix.PNG">

From the original preference workbook, we can read these values from the assignment workbook for a more readable version of the result.

<img src="/assets/images/spl-scheduling/result.PNG">

### Future extensions:
* Generally, desk shifts have higher pay than shelving shifts. Thus, those who can work both may want to be guaranteed a desk shift. If we look at the current schedule, VG and ND are relegated to picking up the left-over shelving shifts.

### Future improvements:
* Eventually migrate to Google Spreadsheet to use preference updating with Google Forms which may facilitate easy creation of a Google Calendar.
