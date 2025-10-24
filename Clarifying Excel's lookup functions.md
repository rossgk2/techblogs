I've decided to write some of my own documentation for common use cases of the Excel functions `LOOKUP`, `VLOOKUP`, `HLOOKUP` and `XLOOKUP` because the official documentation is pretty confusing. It uses "lookup value" as a synonym for "key", when one would conventionally expect a "lookup value" to be a synonym for "value"! (After all, in the typical key-value terminology, "values" are obtained as the result of looking up "keys"!)

Before jumping in- here's a quick overview. Thankfuly, **all four lookup functions essentially return the value coorresponding to the key**. Additionally,

- `LOOKUP` is the most simplistic of the four functions- it pretty much looks up "values" from "keys" like you would expect.
- The "V" and "H" in `VLOOKUP` and `HLOOKUP` stand for "vertical" and "horizontal", respectively; in `VLOOKUP`, the provided 1D ranges must be columns, and in `HLOOKUP` they must be rows.
- `XLOOKUP` combines the functionality of `VLOOKUP` and `XLOOKUP`, and allows for the provided 1D ranges to be either rows or columns. (If you have access to `XLOOKUP`, you should prefer it over `VLOOKUP` and `HLOOKUP`. But at the time of writing, you need access to a Microsoft 365 subscription to use `XLOOKUP`).

Without further ado, here is my documentation.

# `LOOKUP`

Syntax: `LOOKUP(key, keys, values)`.

Returns the result of the pseudocode `values[keys.indexOf(key)]`, where `keys.indexOf(key)` is the index of the `key` in `keys`, when `keys` is treated as an array.

```
key` - a value that exists in `keys
```

`keys` - a 1D range of "keys"

`values` - a 1D range of "values"

Notes:

- The official documentation mentions an "array form" of a `LOOKUP` invocation. I don't cover that here (the above summarizes the "vector form") because `VLOOKUP`, `HLOOKUP`, and `XLOOKUP` accomplish the same thing as the "array form".

 

# `VLOOKUP`

Syntax: `VLOOKUP(key, table, valuesIndex, fuzzyMatch)`.

Returns the result of the pseudocode `values[keys.indexOf(key)]`, where `keys` is the column of "keys", "values" is the column of "values", and where `keys.indexOf(key)` is the index of the `key` in `keys`, when `keys` is treated as an array.

```
key` - a value that exists in `keys
```

`table` - a 2D range that contains the column of "keys" and the column of "values" OR a table that contains the column of "keys" and the column of "values"

`valuesIndex` - the column index (into `table`) of the column of "values"

`fuzzyMatch` - whether or not to fuzzily match key with values in the column of "keys" (you almost always want to use `fuzzyMatch = FALSE`)

Notes:

- To create a table that you would use for the `table` argument, select the 2D range that is to be registered as a table. Then, go to the Insert tab, click Table, and then click OK.
- You might ask: "Why would we want to specify a table that the "key" and "value" columns reside in? Why not just specify the 'key' and 'value' columns?" The reason it's advantageous to have this `table` parameter is that it, if we are calling `HLOOKUP` multiple times in the same column and varying `valuesIndex` between calls, we will get an error message if `valuesIndex` ventures outside the bounds of `table`. This error message can prevent us from making erroneous computations.

# `HLOOKUP`

`HLOOKUP` works in the same way as `VLOOKUP`, with the only difference being that the "keys" and "values" must be stored in rows instead of columns.

# `XLOOKUP`

Syntax: `XLOOKUP(key, keys, values)`.

Returns the result of the pseudocode `values[keys.indexOf(key)]`, where `keys.indexOf(key)` is the index of the `key` in `keys`, when `keys` is treated as an array.

`key` - a value that exists in `keys

`keys` - a 1D range of "keys"

`values` - a 1D range of "values"

Notes:

- A 1D range can be either a row or a column.
