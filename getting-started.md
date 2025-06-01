# Getting started

Using [Elixir](https://elixir-lang.org)'s [Explorer](https://hexdocs.pm/explorer/Explorer.html) 
for the [Polars]() [Getting started](https://docs.pola.rs/user-guide/getting-started/) chapter.

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
