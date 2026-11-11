# Unfinished ML task: tire compound recommendation

There was the M6 machine-learning question assigned during a university course. I did not finish this part in time. The original data files were supplied for the course but were not version-controlled, so the inputs must be recovered or regenerated before this can be implemented and tested :P

## What the assignment asked for

For **one chosen F1 team**, predict the best tire compound for a future race given a circuit and weather conditions such as air temperature and humidity. The five possible answers are **Soft, Medium, Hard, Intermediate, and Wet**. The required algorithm for M6 was **Naive Bayes**. The brief suggested predicting an upcoming race, or a race on the same circuit in a later year while assuming the same teams and drivers.

## Data to recover or regenerate

The current loader expects external files in `raw/output/`, and that directory is absent from a fresh clone. For M6, the essential inputs are:

| Data | Existing expected source | Why it matters |
| --- | --- | --- |
| Per-lap race data and lap times | Files ending in `Race_laps.csv` and `lap_times.csv` | Measure performance and align observations to a race, driver, and lap. |
| Per-lap or time-stamped weather | Files ending in `weather.csv` | Supply air temperature, humidity, and any other available conditions at the time of each observation. |
| Tire compound and stint or pit-stop information | **Source not confirmed**; inspect the recovered lap files first | Identify which of the five compounds was used on each lap or stint. If absent, find or generate a separate tire-usage source. |
| Circuit and race identifiers | `circuits.csv` plus race names/dates in the race files | Match laps and weather to the correct circuit and event. |
| Historical driver-to-team membership | `drivers_teams.csv`; the bundled `raw/api/api.json` has driver and team details | Restrict training and evaluation to the chosen team and join its drivers across races. |

The original assignment described CSV sources for track information, lap and split times, telemetry, race results, and weather, plus text race-control logs and a driver/team API. 

## Work needed

1. **Restore and inspect the data.** Recover the course files or regenerate equivalent raw inputs. Record their source, season coverage, column names, units, and matching keys. Confirm that tire compounds and weather can be aligned with individual laps or stints.
2. **Define “best.”** Choose an explicit training target, such as an adjusted pace or stint-performance measure under comparable conditions. A compound that a team happened to choose is an observed decision, not proof that it was optimal. Document how incomplete races, pit laps, safety cars, and wet conditions affect the label.
3. **Build a training table.** Join tire usage, lap performance, weather, circuit, race, driver, and team data at a consistent granularity. Add dbt staging/intermediate models and data tests as needed. The current loader creates `raw_laps`, `raw_weather`, `raw_circuits`, and `raw_lap_times`, but no dbt models currently use them for M6.
4. **Train a Naive Bayes classifier.** Encode circuit and compound categories, handle numeric weather features, and choose a Naive Bayes variant appropriate for that encoding. Add an ML dependency such as scikit-learn; none is declared in `pyproject.toml` today.
5. **Evaluate as a future-race prediction.** Hold out later races or seasons rather than randomly mixing laps from the same race across training and test sets. Compare against a simple baseline and report per-compound results, especially because Wet and Intermediate examples may be rare.
6. **Show the result.** Add an Evidence page for the chosen team with circuit/weather inputs, the recommended compound, model evaluation, data coverage, and a short explanation of the target and limitations.
