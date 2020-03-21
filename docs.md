# SuperDice documentation

SuperDice is a web-based calculator for dice probabilities. You write a small program that describes how to roll the dice, and it computes all the possible outcomes and their probabilities. It can also draw graphs to display the results.

This is a complete reference documentation. To learn how to use SuperDice, you might want to read the tutorials and examples first.

## Introduction

Programs in SuperDice are written in the Lua programming language. The tool is actually a full-featured Lua 5.3 interpreter that includes a dice probabilities computation library, graphing tools and convenient shortcuts.

You can refer to the [reference manual for Lua 5.3](https://www.lua.org/manual/5.3/) for any questions concerning the language itself.

This documentation describes essentially the dice probabilities library and the environment setup for the web tool, which provides a few global functions, and introduces two object classes: `Die` and `DiceCollection`.

## Automatic conversion

Throughout the library, wherever a `Die` object is expected, one can provide a single value instead (number, string or boolean), and it will be converted to a `Die` object with that single outcome.

Similarly, instead of a `Die`, one can provide a `DiceCollection`, which will be converted to a single `Die` via the `sum` method.

## Global functions

### d(n)

Returns a `Die` object with outcomes 1 to `n`, all equiprobable.

### d(outcomes [, probabilities])

Returns a `Die` object with given `outcomes`, and given relative `probabilities` (defaults to equiprobable). Outcomes can be numbers, string, booleans or other `Die` objects. Different types of outcomes cannot be mixed in a single `Die`.

### dN

All globals `d1`, `d2`, `d3`, etc. (`d` followed by any number of digits) are pre-defined to the same result as `d(N)`.

###  abs, acos, ...

The entire contents of the Lua `math` library are in the global environment: `abs`, `acos`, `asin`, `atan`, `ceil`, `cos`, `deg`, `exp`, `floor`, `fmod`, `huge`, `log`, `max`, `maxinteger`, `min`, `mininteger`, `modf`, `pi`, `rad`, `random`, `randomseed`, `sin`, `sqrt`, `tan`, `tointeger`, `type`, `ult`

### write

Although the `io` library is not available, the `write` function is provided to replace `io.write`, and works similarly.

### plot(die [, label])

Plots a single die: probabilities as a bar chart, and for a die with non boolean outcomes, the two cumulative distributions overlaid as lines. The optional `label` string is used in the plot's legend.

### plot(die1 [, label1], die2 [, label2], ...)

Plots the probabilities for multiple dice in a single plot, as lines. Each die can be followed by an optional string argument that will be used a label in the legend.

### plot_cdf(die1 [, label1], die2 [, label2], ...)

Similar to `plot` for multiple dice, but plots the cumulative distributions instead (probabilities of outcomes lower than or equal). Cannot be used with boolean-valued dice.

Alternatively, the function can take a single table argument containing the dice and labels.

### plot_cdf2(die1 [, label1], die2 [, label2], ...)

Similar to `plot` for multiple dice, but plots the opposite cumulative distributions instead (probabilities of outcomes greater than or equal). Cannot be used with boolean-valued dice.

Alternatively, the function can take a single table argument containing the dice and labels.

### plot_transposed(die1 [, label1], die2 [, label2], ...)

Plots multiple dice so that each die is a column of all its outcomes in a stacked bar chart. This can be useful to visualize and compare dice with a low number of outcomes.

Alternatively, the function can take a single table argument containing the dice and labels.

### plot_raw(labels, datasets, stacked, percentage)

The library exposes the internal plotting function, for maximum flexibility. `labels` is an array of labels (the X axis) and `datasets` is an array containing the datasets to plot. Each dataset should be an array of values (same size as `labels`) and can optionally contain the following fields:

- `label`: the name to use for the dataset in the legend
- `type`: by default the charts are lines, but this can be set to the string `"bar"` instead

If `stacked` is true, the graph will be stacked bars and/or lines, and if `percentage` is true, the Y axis will be displayed as percentages instead of direct values.

## `Die` object

### Die:summary()

Returns a string that summarizes the die: the sorted list of outcomes with their probabilities, as well as the cumulative distributions (probability to be lower or higher than a given outcome).

This is what is also returned when a `Die` is converted to a string using `tostring` (and therefore when using `print`).

### Die:compute_stats()

Returns a table with the following fields:

- `boolean`: whether the die's outcomes are booleans or not
- `outcomes`: the sorted list of possible outcomes of this die
- `probabilities`: the probabilities associated with the outcomes, in the same order
- `lte`: the cumulative distribution, that is, for each outcome, the probability of getting this outcome or a lower one (in the same order as `outcomes`)
- `gte`: the other cumulative distribution: for each outcome, the probability of getting this outcome or a higher one (in the same order as `outcomes`)

Fields `lte` and `gte` are omitted for boolean dice, and the `outcomes` table is not sorted, since booleans cannot be orderd.

### Die:apply(func)

Returns a new `Die` by applying the given function to each outcome. See `DiceCollection:apply()`.

### ..

The concatenation operator is overloaded so that `a .. b` returns a `DiceCollection` made of dice `a` and `b`. If either operand is a number, it is converted to a constant `Die` first. If either operand is a `DiceCollection`, it is converted to a `Die` by computing its sum.

### + - *  / // ^ %

The usual arithmetic operators are overloaded for the `Die` object, and correspond to applying the given operations to the two operands. For instance, `a + b` is equivalent to `(a .. b):apply(function(x,y) return x + y end)`.

The `*` operator is an exception: if the left-hand side operand is a number N, the result is instead a `DiceCollection` containing N repetitions of the right-hand side operand.

The `-` operator also works as the unary operator for negation.

Due to limitations in operator overloading in Lua, the comparisons functions are available as methods instead:

- `lt` for operator `<`
- `lte` for operator `<=`
- `gt` for operator `>`
- `gte` for operator `>=`
- `eq` for operator `==`
- `neq` for operator `~=`

so that for instance `a:lt(b)` is equivalent to `(a .. b):apply(function(x,y) return x < y end)`.

### Die(outcome)

A `Die` object can be called like a function to get the probability of the given `outcome`. This will of course return 0 if the outcome is not possible on this die.

## `DiceCollection` object