# Getting started

Using [Elixir](https://elixir-lang.org)'s [Explorer](https://hexdocs.pm/explorer/Explorer.html) 
for the [Polars](https://pola.rs)'s 
[Getting started](https://docs.pola.rs/user-guide/getting-started/) chapter.

The [Ten Minutes to Explorer](https://hexdocs.pm/explorer/exploring_explorer.html) 
introduction to Explorer is the first starting place.

I'm always interested in learning better ways to use the Explorer API, 
so feedback is welcome.

## Setup

Installation/setup is easy and fast. Create some aliases.

```elixir
Mix.install([:explorer])

alias Explorer.DataFrame
alias Explorer.Series
require DataFrame, as: DF
```

## Reading & writing

Creating a DataFrame from Elixir code.

```elixir
iex> df = DF.new(
  name: ["Alice Archer", "Ben Brown", "Chloe Cooper", "Daniel Donovan"],
  birthday: [~D[1997-01-10], ~D[1985-02-15], ~D[1982-03-22], ~D[1981-04-30]],
  weight: [57.9, 72.5, 53.6, 83.1],
  height: [1.56, 1.77, 1.65, 1.75])

#Explorer.DataFrame<
  Polars[4 x 4]
  name string ["Alice Archer", "Ben Brown", "Chloe Cooper", "Daniel Donovan"]
  birthday date [1997-01-10, 1985-02-15, 1982-03-22, 1981-04-30]
  weight f64 [57.9, 72.5, 53.6, 83.1]
  height f64 [1.56, 1.77, 1.65, 1.75]
>

iex> DF.print(df)
```
```
+-----------------------------------------------+
|   Explorer DataFrame: [rows: 4, columns: 4]   |
+----------------+------------+--------+--------+
|      name      |  birthday  | weight | height |
|    <string>    |   <date>   | <f64>  | <f64>  |
+================+============+========+========+
| Alice Archer   | 1997-01-10 | 57.9   | 1.56   |
+----------------+------------+--------+--------+
| Ben Brown      | 1985-02-15 | 72.5   | 1.77   |
+----------------+------------+--------+--------+
| Chloe Cooper   | 1982-03-22 | 53.6   | 1.65   |
+----------------+------------+--------+--------+
| Daniel Donovan | 1981-04-30 | 83.1   | 1.75   |
+----------------+------------+--------+--------+
```

Writing to and reading from CSV.

```elixir
DF.to_csv(df, "/tmp/output.csv")

DF.from_csv("/tmp/output.csv", parse_dates: true)
```

## Expressions & contexts

### Select/mutate

Mutation and selection are separate operations.

Note how the expressions defining the new columns can be written as code,
provided you use supported/recognized operations.

```elixir
iex> result = DF.mutate(
  df, 
  birth_year: year(birthday), 
  bmi: weight / (height * height))

#Explorer.DataFrame<
  Polars[4 x 6]
  name string ["Alice Archer", "Ben Brown", "Chloe Cooper", "Daniel Donovan"]
  birthday date [1997-01-10, 1985-02-15, 1982-03-22, 1981-04-30]
  weight f64 [57.9, 72.5, 53.6, 83.1]
  height f64 [1.56, 1.77, 1.65, 1.75]
  birth_year s32 [1997, 1985, 1982, 1981]
  bmi f64 [23.791913214990135, 23.14149829231702, 19.687786960514234,
   27.13469387755102]

iex> result = DF.select(result, [:name, :birth_year, :bmi])

#Explorer.DataFrame<
  Polars[4 x 3]
  name string ["Alice Archer", "Ben Brown", "Chloe Cooper", "Daniel Donovan"]
  birth_year s32 [1997, 1985, 1982, 1981]
  bmi f64 [23.791913214990135, 23.14149829231702, 19.687786960514234,
   27.13469387755102]
>

iex> DF.print(result)
```
```
+--------------------------------------------------+
|    Explorer DataFrame: [rows: 4, columns: 3]     |
+----------------+------------+--------------------+
|      name      | birth_year |        bmi         |
|    <string>    |   <s32>    |       <f64>        |
+================+============+====================+
| Alice Archer   | 1997       | 23.791913214990135 |
+----------------+------------+--------------------+
| Ben Brown      | 1985       | 23.14149829231702  |
+----------------+------------+--------------------+
| Chloe Cooper   | 1982       | 19.687786960514234 |
+----------------+------------+--------------------+
| Daniel Donovan | 1981       | 27.13469387755102  |
+----------------+------------+--------------------+
```

Of course both operations can be combined in typical Elixir fashion.

```elixir
df
|> DF.mutate(
  birth_year: year(birthday), 
  bmi: weight / (height * height))
|> DF.select([:name, :birth_year, :bmi])
```

Using the keep option with mutate.

```elixir
iex> result = DF.mutate(
  df, 
  [
    name: name, 
    "weight-5%": round(weight * 0.95, 2), 
    "height-5%": round(height * 0.95, 2)], 
  keep: :none)

#Explorer.DataFrame<
  Polars[4 x 3]
  name string ["Alice Archer", "Ben Brown", "Chloe Cooper", "Daniel Donovan"]
  weight-5% f64 [55.01, 68.88, 50.92, 78.94]
  height-5% f64 [1.48, 1.68, 1.57, 1.66]
>

iex> DF.print(result)
```
```
+-------------------------------------------+
| Explorer DataFrame: [rows: 4, columns: 3] |
+-----------------+------------+------------+
|      name       | weight-5%  | height-5%  |
|    <string>     |   <f64>    |   <f64>    |
+=================+============+============+
| Alice Archer    | 55.01      | 1.48       |
+-----------------+------------+------------+
| Ben Brown       | 68.88      | 1.68       |
+-----------------+------------+------------+
| Chloe Cooper    | 50.92      | 1.57       |
+-----------------+------------+------------+
| Daniel Donovan  | 78.94      | 1.66       |
+-----------------+------------+------------+
```

### Filter

Simple filter.

Again we can write almost regular code.

```elixir
iex> result = DF.filter(df, year(birthday) < 1990)

#Explorer.DataFrame<
  Polars[3 x 4]
  name string ["Ben Brown", "Chloe Cooper", "Daniel Donovan"]
  birthday date [1985-02-15, 1982-03-22, 1981-04-30]
  weight f64 [72.5, 53.6, 83.1]
  height f64 [1.77, 1.65, 1.75]
>

iex> DF.print(result)
```
```
+-----------------------------------------------+
|   Explorer DataFrame: [rows: 3, columns: 4]   |
+----------------+------------+--------+--------+
|      name      |  birthday  | weight | height |
|    <string>    |   <date>   | <f64>  | <f64>  |
+================+============+========+========+
| Ben Brown      | 1985-02-15 | 72.5   | 1.77   |
+----------------+------------+--------+--------+
| Chloe Cooper   | 1982-03-22 | 53.6   | 1.65   |
+----------------+------------+--------+--------+
| Daniel Donovan | 1981-04-30 | 83.1   | 1.75   |
+----------------+------------+--------+--------+
```

Multiple conditions in a filter. The list is a conjunction.

Sigils are pretty handy here.

```elixir
iex> result = DF.filter(
  df, 
  [
    ~D[1982-12-31] <= birthday,
    birthday <= ~D[1996-01-01],
    height > 1.7])

#Explorer.DataFrame<
  Polars[1 x 4]
  name string ["Ben Brown"]
  birthday date [1985-02-15]
  weight f64 [72.5]
  height f64 [1.77]
>

iex> DF.print(result)
```
```
+----------------------------------------------+
|  Explorer DataFrame: [rows: 1, columns: 4]   |
+------------+-------------+---------+---------+
|    name    |  birthday   | weight  | height  |
|  <string>  |   <date>    |  <f64>  |  <f64>  |
+============+=============+=========+=========+
| Ben Brown  | 1985-02-15  | 72.5    | 1.77    |
+------------+-------------+---------+---------+
```

### Group_by

Three elementary operations in row.

```elixir
iex> result = (df 
  |> DF.mutate(decade: quotient(year(birthday), 10) * 10)
  |> DF.group_by([:decade])
  |> DF.summarise(len: count(decade)))

#Explorer.DataFrame<
  Polars[2 x 2]
  decade s64 [1990, 1980]
  len u32 [1, 3]
>

iex> DF.print(result)
```
```
+--------------------------------------------+
| Explorer DataFrame: [rows: 2, columns: 2]  |
+----------------------+---------------------+
|        decade        |         len         |
|        <f64>         |        <u32>        |
+======================+=====================+
| 1990                 | 1                   |
+----------------------+---------------------+
| 1980                 | 3                   |
+----------------------+---------------------+
```

Now with more aggregations.

```elixir
iex> result = (df 
  |> DF.mutate(decade: quotient(year(birthday), 10) * 10)
  |> DF.group_by([:decade])
  |> DF.summarise(
    len: count(decade), 
    avg_weight: round(mean(weight), 2), 
    tallest: max(height)))

#Explorer.DataFrame<
  Polars[2 x 4]
  decade s64 [1990, 1980]
  len u32 [1, 3]
  avg_weight f64 [57.9, 69.73]
  tallest f64 [1.56, 1.77]
>

iex> DF.print(result)
```
```
+-------------------------------------------+
| Explorer DataFrame: [rows: 2, columns: 4] |
+---------+--------+-------------+----------+
| decade  |  len   | avg_weight  | tallest  |
|  <f64>  | <u32>  |    <f64>    |  <f64>   |
+=========+========+=============+==========+
| 1990    | 1      | 57.9        | 1.56     |
+---------+--------+-------------+----------+
| 1980    | 3      | 69.73       | 1.77     |
+---------+--------+-------------+----------+
```

## More complex queries

Though split works, selecting the first element with at does not seem possible.
By using split_into a structure and then accessing a field we get the same result.

```elixir
iex> result = (df 
  |> DF.mutate(
    name: field(split_into(name, " ", [:first, :last]), :first), 
    decade: quotient(year(birthday), 10) * 10) 
  |> DF.group_by([:decade]) 
  |> DF.summarise(
    name: name, 
    avg_weight: round(mean(weight), 2), 
    avg_height: round(mean(height), 2)))

#Explorer.DataFrame<
  Polars[2 x 4]
  decade s64 [1990, 1980]
  name list[string] [["Alice"], ["Ben", "Chloe", "Daniel"]]
  avg_weight f64 [57.9, 69.73]
  avg_height f64 [1.56, 1.72]
>

iex> DF.print(result)
```
```
+---------------------------------------------------+
|     Explorer DataFrame: [rows: 2, columns: 4]     |
+--------+----------------+------------+------------+
| decade |      name      | avg_weight | avg_height |
| <f64>  | <list[string]> |   <f64>    |   <f64>    |
+========+================+============+============+
| 1990   | [Alice]        | 57.9       | 1.56       |
+--------+----------------+------------+------------+
| 1980   | [              | 69.73      | 1.72       |
|        |  Ben           |            |            |
|        |  Chloe         |            |            |
|        |  Daniel        |            |            |
|        | ]              |            |            |
+--------+----------------+------------+------------+
```

## Combining dataframes

### Joining dataframes

```elixir
iex> df2 = DataFrame.new(
  name: ["Ben Brown", "Daniel Donovan", "Alice Archer", "Chloe Cooper"],
  parent: [true, false, false, false],
  siblings: [1, 2, 3, 4])

iex> result = DF.join(df, df2, on: :name, how: :left)

#Explorer.DataFrame<
  Polars[4 x 6]
  name string ["Alice Archer", "Ben Brown", "Chloe Cooper", "Daniel Donovan"]
  birthday date [1997-01-10, 1985-02-15, 1982-03-22, 1981-04-30]
  weight f64 [57.9, 72.5, 53.6, 83.1]
  height f64 [1.56, 1.77, 1.65, 1.75]
  parent boolean [false, true, false, false]
  siblings s64 [3, 1, 4, 2]
>

iex> DF.print(result)
```
```
+----------------------------------------------------------------------+
|              Explorer DataFrame: [rows: 4, columns: 6]               |
+----------------+------------+--------+--------+-----------+----------+
|      name      |  birthday  | weight | height |  parent   | siblings |
|    <string>    |   <date>   | <f64>  | <f64>  | <boolean> |  <s64>   |
+================+============+========+========+===========+==========+
| Alice Archer   | 1997-01-10 | 57.9   | 1.56   | false     | 3        |
+----------------+------------+--------+--------+-----------+----------+
| Ben Brown      | 1985-02-15 | 72.5   | 1.77   | true      | 1        |
+----------------+------------+--------+--------+-----------+----------+
| Chloe Cooper   | 1982-03-22 | 53.6   | 1.65   | false     | 4        |
+----------------+------------+--------+--------+-----------+----------+
| Daniel Donovan | 1981-04-30 | 83.1   | 1.75   | false     | 2        |
+----------------+------------+--------+--------+-----------+----------+
```

### Concatenating dataframes

There is concat_rows and concat_columns.

```elixir
iex> df3 = DataFrame.new(
  name: ["Ethan Edwards", "Fiona Foster", "Grace Gibson", "Henry Harris"],
  birthday: [~D[1977-05-10], ~D[1975-06-23], ~D[1973-07-22], ~D[1971-08-03]],
  weight: [67.9, 72.5, 57.6, 93.1],
  height: [1.76, 1.6, 1.66, 1.8])

#Explorer.DataFrame<
  Polars[4 x 4]
  name string ["Ethan Edwards", "Fiona Foster", "Grace Gibson", "Henry Harris"]
  birthday date [1977-05-10, 1975-06-23, 1973-07-22, 1971-08-03]
  weight f64 [67.9, 72.5, 57.6, 93.1]
  height f64 [1.76, 1.6, 1.66, 1.8]
>

iex> DF.concat_rows(df, df3)

#Explorer.DataFrame<
  Polars[8 x 4]
  name string ["Alice Archer", "Ben Brown", "Chloe Cooper", "Daniel Donovan",
   "Ethan Edwards", ...]
  birthday date [1997-01-10, 1985-02-15, 1982-03-22, 1981-04-30, 1977-05-10,
   ...]
  weight f64 [57.9, 72.5, 53.6, 83.1, 67.9, ...]
  height f64 [1.56, 1.77, 1.65, 1.75, 1.76, ...]
>

iex> DF.print(v(), limit: :infinity)
```
```
+-----------------------------------------------+
|   Explorer DataFrame: [rows: 8, columns: 4]   |
+----------------+------------+--------+--------+
|      name      |  birthday  | weight | height |
|    <string>    |   <date>   | <f64>  | <f64>  |
+================+============+========+========+
| Alice Archer   | 1997-01-10 | 57.9   | 1.56   |
+----------------+------------+--------+--------+
| Ben Brown      | 1985-02-15 | 72.5   | 1.77   |
+----------------+------------+--------+--------+
| Chloe Cooper   | 1982-03-22 | 53.6   | 1.65   |
+----------------+------------+--------+--------+
| Daniel Donovan | 1981-04-30 | 83.1   | 1.75   |
+----------------+------------+--------+--------+
| Ethan Edwards  | 1977-05-10 | 67.9   | 1.76   |
+----------------+------------+--------+--------+
| Fiona Foster   | 1975-06-23 | 72.5   | 1.6    |
+----------------+------------+--------+--------+
| Grace Gibson   | 1973-07-22 | 57.6   | 1.66   |
+----------------+------------+--------+--------+
| Henry Harris   | 1971-08-03 | 93.1   | 1.8    |
+----------------+------------+--------+--------+
```

The standard print limit is 5.
