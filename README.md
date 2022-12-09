Ecto Adapter for ClickHouse using [`:ch`](https://github.com/ruslandoga/ch)

```elixir
iex> Mix.install([{:chto, github: "ruslandoga/chto"}])

iex> defmodule Repo do
       use Ecto.Repo, adapter: Ecto.Adapters.ClickHouse, otp_app: :example
     end

iex> import Ecto.Query
iex> Repo.start_link()

# see :ch (linked above) for more "raw query" examples
iex> Repo.query("CREATE TABLE example(a UInt32, b String, c DateTime) engine=Memory")
{:ok, %{num_rows: 0, rows: []}}

# WIP will probably be Repo.insert_stream or Repo.insert_all
iex> rows = [[1, "1", ~N[2022-11-26 09:38:24]], [2, "2", ~N[2022-11-26 09:38:25]], [3, "3", ~N[2022-11-26 09:38:26]]]
iex> Chto.insert_stream(Repo, "example", rows, fields: [:a, :b, :c], types: [:u32, :string, :datetime])
# INSERT INTO "example"("a","b","c") [[1, "1", ~N[2022-11-26 09:38:24]], [2, "2", ~N[2022-11-26 09:38:25]], [3, "3", ~N[2022-11-26 09:38:26]]]
{:ok, _written_rows = 3}

# WIP type(..) will be unnecessary
iex> min_a = 1
iex> "example" |> where([e], e.a > type(^min_a, :integer)) |> select([e], map(e, [:b, :c])) |> Repo.all()
# SELECT e0."b",e0."c" FROM "example" AS e0 WHERE (e0."a" > {$0:Int64}) [1]
[%{b: "2", c: ~N[2022-11-26 09:38:25]}, %{b: "3", c: ~N[2022-11-26 09:38:26]}]

iex> File.write!("example.csv", "a,b,c\n1,1,2022-11-26 09:38:24\n2,2,2022-11-26 09:38:25\n3,3,2022-11-26 09:38:26")
iex> Chto.insert_stream(Repo, "example", File.stream!("example.csv"), format: "CSVWithNames")
# INSERT INTO "example" %File.Stream{path: "example.csv", modes: [:raw, :read_ahead, :binary], line_or_bytes: :line, raw: true}
{:ok, _written_rows = 3}

iex> File.rm!("example.csv")
iex> Repo.query("DROP TABLE example")
```
