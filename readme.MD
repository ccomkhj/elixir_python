# Livebook sample project: Elixir in Data Science

Data processing is often dominated by Python and libraries like pandas and spark. However, Elixir is emerging as a strong contender. By using Elixir throughout your application stack including the data processing, you enhance code reusability and maintainability. This repo explores how to 
- leverage Elixir's capabilities for data frame manipulations using [livebook](https://github.com/livebook-dev/livebook).
- show the foundation to access database using Ecto.

## How to start?
1. Simply run

```bash
docker compose up
```

2. To run pheonix demo project, follow below
```bash
cd demo_app
iex --name test@0.0.0.0 --cookie hexacookie -S mix phx.server
```
name: `test@0.0.0.0`
cookie secret: `hexacookie`

3. Open samples under `data` (default) directory. You will discover the elixir-livebook sample projects.
