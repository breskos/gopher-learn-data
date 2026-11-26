# gopher-learn-data

Data for the examples, tests and benchmarks of [gopher-learn](https://github.com/breskos/gopher-learn).

## Example data

Every example of gopher-learn has a folder with the same name here:

| Folder | Example |
| --- | --- |
| [sonar](sonar) | Classification of sonar signals (mines vs. rocks) |
| [online-sonar](online-sonar) | The sonar data streamed into the online learner |
| [wine-quality](wine-quality) | Regression of wine quality |
| [ngram](ngram), [dictionary](dictionary) | Classification of German questions by answer type |

## Synthetic data

The [synthetic](synthetic) folder holds generated data sets with a known structure, so you can tell how well a model should do. They contain no third-party data. The folders are named after what the data is for:

| Folder | Use it for | Data sets |
| --- | --- | --- |
| [classification](synthetic/classification) | Batch classification with `engine` | blobs, imbalanced, spirals, high-dim, predictive maintenance, activity recognition, Wi-Fi localization |
| [regression](synthetic/regression) | Batch regression with `engine` | Friedman #1, unscaled features, run-to-failure, air quality calibration, smart meter |
| [online](synthetic/online) | Streams for `online`, read in order | abrupt, gradual, recurring and incremental drift, room occupancy, sensor health, aging air quality sensor |
| [service](synthetic/service) | Event logs for the `service` package: delayed, wrong and corrupt feedback, corrections, drift | offer acceptance, delivery times |
| [text](synthetic/text) | String `encoders` | German intent phrases |
| [loading](synthetic/loading) | Loader edge cases | semicolon CSV, missing values, sparse LIBSVM |
| [benchmark](synthetic/benchmark) | Performance measurements | 250,000 rows × 20 features |

Sensor data sets (predictive maintenance, activity, room occupancy, air quality, …) are modeled on common sensors with simple physical and behavioral models; they look like sensor data but are not measurements. [synthetic/README.md](synthetic/README.md) describes every set, its columns and how to load it.

The folder is also a small Go module (standard library only) with a catalog, a reader for every file format and examples. To check all files:

```
cd synthetic && go test ./...
```

The files are reproducible. In the gopher-learn repository, run:

```
go run ./internal/cmd/gendata -out ../gopher-learn-data/synthetic
```
