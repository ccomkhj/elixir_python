# Transitioning from Python to Elixir in Data Science

Data processing is often dominated by Python and libraries like pandas and spark. However, Elixir is emerging as a strong contender. By using Elixir throughout your application stack including the data processing, you enhance code reusability and maintainability. This blog post explores how to leverage Elixir's capabilities for data frame manipulations traditionally performed in Python, showcasing its benefits in data-intensive production scenarios.

## Getting Started with Elixir for Data Science

To emulate Python's data manipulation capabilities, we first need to set up our Elixir environment with the necessary libraries. Here's how you can do it:

```elixir
Mix.install(
  [
    {:kino_explorer, "~> 0.1.20"},
    {:explorer, "~> 0.10.0"},
    {:httpoison, "~> 1.8"},
    {:jason, "~> 1.4"},
    {:vega_lite, "~> 0.1.7"},
    {:exla, ">= 0.0.0"},
    {:kino_vega_lite, "~> 0.1.11"}
  ],
  config: [nx: [default_backend: EXLA.Backend]]
)
```

- **Mix.Install/2**: This installs the necessary Elixir libraries for data manipulation and visualization. Here, `Explorer` is used for data frames, similar to pandas in Python.

## Basic DataFrame Manipulation

Let's dive into a simple data manipulation task. In Python, you might double the values in a column with `df["feature"] = df["feature"] * 2`. Here's how we achieve the same in Elixir:

```elixir
alias Explorer.DataFrame
alias Explorer.Series

# Sample data frame creation
data = %{feature: [1, 2, 3, 4, 5]}

# Creating the DataFrame
df = DataFrame.new(data)

# Doubling the "feature" column values
doubled_feature = DataFrame.pull(df, "feature")
  |> Series.multiply(2)

# Updating the DataFrame
df = DataFrame.put(df, "feature", doubled_feature)

IO.inspect(df)
```

Here, we seamlessly manipulate data frames using Elixir's functional paradigms, highlighting an expressive approach similar to Python.

## Advanced Data Processing

Handling missing values is a common data science task. Let's perform **linear interpolation**, a bit more involved in Elixir:

```elixir
defmodule DataFrameProcessor do
  # Interpolating missing values in a series
  def interpolate(series) do
    values = Series.to_list(series)

    interpolated = Enum.reduce_while(values, [], fn value, acc ->
      case value do
        nil ->
          last_value = List.last(acc) || 0.0
          index = length(acc)
          rest = Enum.drop(values, index + 1)
          next_value = Enum.find(rest, fn x -> not is_nil(x) end)
          interpolated_value = if not is_nil(next_value), do: (last_value + next_value) / 2, else: last_value
          {:cont, acc ++ [interpolated_value]}
        _ ->
          {:cont, acc ++ [value]}
      end
    end)

    Series.from_list(interpolated)
  end

  def process_and_join(date_list, feature1, feature2) do
    data_df1 = %{"feature1" => feature1, "date" => date_list}
    data_df2 = %{"feature2" => feature2, "date" => date_list}

    df1 = DataFrame.new(data_df1) |> DataFrame.put("feature1", interpolate(DataFrame.pull(df1, "feature1")))
    df2 = DataFrame.new(data_df2) |> DataFrame.put("feature2", interpolate(DataFrame.pull(df2, "feature2")))

    DataFrame.join(df1, df2, on: "date", how: :left)
  end
end

# Sample data for testing
date_list = [~D[2023-01-01], ~D[2023-01-02], ~D[2023-01-03], ~D[2023-01-04], ~D[2023-01-05]]
feature1 = [1.0, nil, 3.0, nil, 5.0]
feature2 = [nil, 8.0, nil, 16.0, 20.0]

DataFrameProcessor.process_and_join(date_list, feature1, feature2)
```

The logging used in this module provides insights into the interpolation process, aiding debugging and understanding.

## Calculating Moving Averages

Moving averages offer insights into data trends. Here’s how to calculate them in Elixir:

```elixir
defmodule TemperatureAnalysis do
  alias Explorer.DataFrame
  alias Explorer.Series

  def moving_average(date_list, temperature_list, window_size, weights \\ []) do
    data = %{"date" => date_list, "temperature" => temperature_list}
    df = DataFrame.new(data)

    moving_avg_series = Series.window_mean(DataFrame.pull(df, "temperature"), window_size, weights: weights, min_periods: 1)
    DataFrame.put(df, "moving_average", moving_avg_series)
  end
end

defmodule DateGenerator do
  def generate_dates(start_date, days) do
    Enum.map(0..(days-1), &Date.add(start_date, &1))
  end
end

# Sample Input
date_list = DateGenerator.generate_dates(~D[2023-01-01], 21)
temperature_list = Enum.map(1..21, fn day -> 10.0 * :math.sin(day / 2) + Enum.random(-2..2) + 20 end)

TemperatureAnalysis.moving_average(date_list, temperature_list, 3)
```

This segment illustrates Elixir’s versatility in maintaining structure while providing computational insights.

## Visualization with VegaLite

Visualization in Elixir can be basic but effective using VegaLite. Here's a simple example:

```elixir
VegaLite.new(width: 1080, title: "Temperature Analysis")
|> VegaLite.data_from_values(temperature, only: ["date", "moving_average", "temperature"])
|> VegaLite.layers([
  VegaLite.new() |> VegaLite.mark(:line) |> VegaLite.encode_field(:x, "date", type: :temporal) |> VegaLite.encode_field(:y, "moving_average", type: :quantitative),
  VegaLite.new() |> VegaLite.mark(:point) |> VegaLite.encode_field(:x, "date", type: :temporal) |> VegaLite.encode_field(:y, "temperature", type: :quantitative)
])
```

While the visualization capabilities are still evolving, Elixir provides a foundation for basic analytical plots.

## Conclusion

Elixir, with its focus on concurrency and reliability, offers a refreshing perspective on data frame manipulation in data science. While it may not yet match the breadth of libraries available in Python, its potential, especially in applications needing robust backend processing, is significant. Explore these possibilities in your projects and consider contributing to their development.