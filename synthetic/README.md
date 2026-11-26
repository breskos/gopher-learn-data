# Synthetic data sets for gopher-learn

All data in this folder is generated; it contains no third-party data. Every data set has a known structure, so you can tell how well a model should do. Data sets marked *sensor* model real sensors with simple physical and behavioral models; they look like sensor data but are not measurements.

The folders are named after what the data is for:

| Folder | Purpose |
| --- | --- |
| [classification](#classification) | Batch classification: train with engine.New and evaluate on a held-out split. |
| [regression](#regression) | Batch regression with targets in [0, 1]: train with engine.New, standardize the features with learn.FitScaler. |
| [online](#online) | Streams for online learning: read the rows in order and feed them to online.Online one by one. The concept column (where present) tells which concept generated a row; ignore it for training and use it to evaluate drift detection. |
| [service](#service) | Event logs for the service package: every row is a prediction request whose feedback arrives feedback_delay seconds later, is sometimes wrong and is then corrected correction_delay seconds after it. Replay them in time order; the concept column marks the drift. |
| [text](#text) | Short texts for the string encoders (encoders.New with dictionary, split dictionary or n-grams). |
| [loading](#loading) | Edge cases for the loaders: delimiters, ignored columns, missing values and sparse LIBSVM files. |
| [benchmark](#benchmark) | Larger files for performance measurements. |

## Check and read the data in Go

This folder is a Go module (`github.com/breskos/gopher-learn-data/synthetic`, standard library only). [catalog.go](catalog.go) lists every file with its columns and purpose, [reader.go](reader.go) reads any of them, [example_test.go](example_test.go) shows one reader per folder, and [synthetic_test.go](synthetic_test.go) checks every file:

```
go test ./...
```

The files are reproducible: in the [gopher-learn](https://github.com/breskos/gopher-learn) repository run

```
go run ./internal/cmd/gendata -out <this folder>
```

The same seed produces identical files.

## classification

Batch classification: train with engine.New and evaluate on a held-out split.

| File | Task | Rows | Kind |
| --- | --- | --- | --- |
| [classification/blobs.csv](classification/blobs.csv) | classification | 20000 | benchmark |
| [classification/imbalanced.csv](classification/imbalanced.csv) | classification | 50000 | benchmark |
| [classification/spirals.csv](classification/spirals.csv) | classification | 10000 | benchmark |
| [classification/high-dim.csv](classification/high-dim.csv) | classification | 10000 | benchmark |
| [classification/predictive-maintenance.csv](classification/predictive-maintenance.csv) | classification | 20000 | sensor |
| [classification/activity.csv](classification/activity.csv) | classification | 30000 | sensor |
| [classification/wifi-localization.csv](classification/wifi-localization.csv) | classification | 20000 | sensor |

With gopher-learn:

```go
data := learn.NewSet(nn.Classification)
err := data.LoadCSV("classification/activity.csv",
	learn.WithTarget("activity"), learn.WithIgnore("user"))
// split, then train with engine.New(train, []int{24}, engine.DefaultConfig())
```

### classification/blobs.csv

Four Gaussian clusters (standard deviation 1) in 8 dimensions with centers drawn uniformly from [-5, 5]. Classes are balanced and well separated, so a small network should reach close to 100 % accuracy.

Columns: `x1`, `x2`, `x3`, `x4`, `x5`, `x6`, `x7`, `x8`, `label`. Target: `label`.

### classification/imbalanced.csv

Binary classification with 5 % positives. Features x1–x5 are shifted by +1.5 for positives, x6–x10 are noise. The id column must be ignored. Plain accuracy is misleading here (95 % by always predicting "neg"); use balanced accuracy or F1.

Columns: `id`, `x1`, `x2`, `x3`, `x4`, `x5`, `x6`, `x7`, `x8`, `x9`, `x10`, `label`. Target: `label`. Ignore: `id`.

### classification/spirals.csv

Three interleaved spirals in 2D with small noise. The classes are not linearly separable, so this needs hidden layers and enough training; a good network reaches more than 90 % accuracy.

Columns: `x`, `y`, `label`. Target: `label`.

### classification/high-dim.csv

Binary classification with 200 features: x1–x10 are informative (the label is the sign of a weighted sum plus noise), x11–x30 are noisy copies of x1–x10 (redundant, highly correlated), x31–x200 are pure noise. Useful for the float reducer and for feature importance.

Columns: `x1`, `x2`, `x3`, `x4`, `x5`, `x6`, `x7`, `x8`, `x9`, `x10`, `x11`, `x12`, `x13`, `x14`, `x15`, `x16`, `x17`, `x18`, `x19`, `x20`, `x21`, `x22`, `x23`, `x24`, `x25`, `x26`, `x27`, `x28`, `x29`, `x30`, `x31`, `x32`, `x33`, `x34`, `x35`, `x36`, `x37`, `x38`, `x39`, `x40`, `x41`, `x42`, `x43`, `x44`, `x45`, `x46`, `x47`, `x48`, `x49`, `x50`, `x51`, `x52`, `x53`, `x54`, `x55`, `x56`, `x57`, `x58`, `x59`, `x60`, `x61`, `x62`, `x63`, `x64`, `x65`, `x66`, `x67`, `x68`, `x69`, `x70`, `x71`, `x72`, `x73`, `x74`, `x75`, `x76`, `x77`, `x78`, `x79`, `x80`, `x81`, `x82`, `x83`, `x84`, `x85`, `x86`, `x87`, `x88`, `x89`, `x90`, `x91`, `x92`, `x93`, `x94`, `x95`, `x96`, `x97`, `x98`, `x99`, `x100`, `x101`, `x102`, `x103`, `x104`, `x105`, `x106`, `x107`, `x108`, `x109`, `x110`, `x111`, `x112`, `x113`, `x114`, `x115`, `x116`, `x117`, `x118`, `x119`, `x120`, `x121`, `x122`, `x123`, `x124`, `x125`, `x126`, `x127`, `x128`, `x129`, `x130`, `x131`, `x132`, `x133`, `x134`, `x135`, `x136`, `x137`, `x138`, `x139`, `x140`, `x141`, `x142`, `x143`, `x144`, `x145`, `x146`, `x147`, `x148`, `x149`, `x150`, `x151`, `x152`, `x153`, `x154`, `x155`, `x156`, `x157`, `x158`, `x159`, `x160`, `x161`, `x162`, `x163`, `x164`, `x165`, `x166`, `x167`, `x168`, `x169`, `x170`, `x171`, `x172`, `x173`, `x174`, `x175`, `x176`, `x177`, `x178`, `x179`, `x180`, `x181`, `x182`, `x183`, `x184`, `x185`, `x186`, `x187`, `x188`, `x189`, `x190`, `x191`, `x192`, `x193`, `x194`, `x195`, `x196`, `x197`, `x198`, `x199`, `x200`, `label`. Target: `label`.

### classification/predictive-maintenance.csv

Condition monitoring of electric motors with an accelerometer at the bearing (e.g. ADXL345), a temperature probe and the speed. Each row is one window: overall RMS and peak acceleration, crest factor, kurtosis and the spectral amplitudes at the rotation frequency (1x), twice the rotation frequency (2x) and the bearing outer race fault frequency (BPFO). Imbalance raises 1x, misalignment raises 2x and the temperature, bearing faults raise BPFO, kurtosis and crest factor. Fault severity varies, so mild faults overlap with normal operation. The machine column is an ID and must be ignored.

Columns: `machine`, `rpm`, `temperature_c`, `rms_g`, `peak_g`, `crest_factor`, `kurtosis`, `amp_1x_g`, `amp_2x_g`, `amp_bpfo_g`, `state`. Target: `state`. Ignore: `machine`.

### classification/activity.csv

Human activity recognition with a smartphone in the trouser pocket (3-axis accelerometer in g and gyroscope in rad/s, e.g. MPU-6050, 50 Hz). Each row summarizes a 2.56 s window: mean and standard deviation per axis, the standard deviation of the magnitude, the dominant step frequency and the gyroscope activity. Seven activities: walking, running, upstairs, downstairs, cycling, sitting, standing. 30 users differ in step frequency, intensity and how the phone sits in the pocket, so walking and the stairs overlap. The user column must be ignored; for a fair evaluation split by user.

Columns: `user`, `acc_mean_x`, `acc_mean_y`, `acc_mean_z`, `acc_std_x`, `acc_std_y`, `acc_std_z`, `acc_mag_std`, `acc_dom_freq_hz`, `gyro_std_x`, `gyro_std_y`, `gyro_std_z`, `gyro_mag_mean`, `activity`. Target: `activity`. Ignore: `user`.

### classification/wifi-localization.csv

Indoor localization from Wi-Fi signal strength. An office floor of 40 m x 20 m has eight rooms (R1–R8, a 4 x 2 grid) and six access points. The received signal strength follows a log-distance path loss model (-35 dBm at 1 m, exponent 2.8), loses 6 dB per wall crossed, varies with log-normal shadowing (4 dB) and differs by up to ±4 dB between the five device models. Signals below -95 dBm are reported as -100. The device column must be ignored.

Columns: `device`, `ap1_dbm`, `ap2_dbm`, `ap3_dbm`, `ap4_dbm`, `ap5_dbm`, `ap6_dbm`, `room`. Target: `room`. Ignore: `device`.

## regression

Batch regression with targets in [0, 1]: train with engine.New, standardize the features with learn.FitScaler.

| File | Task | Rows | Kind |
| --- | --- | --- | --- |
| [regression/friedman1.csv](regression/friedman1.csv) | regression | 20000 | benchmark |
| [regression/unscaled.csv](regression/unscaled.csv) | regression | 20000 | benchmark |
| [regression/run-to-failure.csv](regression/run-to-failure.csv) | regression | 20000 | sensor |
| [regression/air-quality-calibration.csv](regression/air-quality-calibration.csv) | regression | 8760 | sensor |
| [regression/smart-meter.csv](regression/smart-meter.csv) | regression | 17520 | sensor |

With gopher-learn:

```go
data := learn.NewSet(nn.Regression)
err := data.LoadCSV("regression/friedman1.csv", learn.WithTarget("y"))
train, test, _ := data.Split(0.8, nil)
scaler, _ := learn.FitScaler(train, learn.Standardize) // fit on training data only
train, _ = scaler.TransformSet(train)
test, _ = scaler.TransformSet(test)
```

### regression/friedman1.csv

The Friedman #1 benchmark: x1–x10 uniform in [0, 1], y = 10 sin(pi x1 x2) + 20 (x3 - 0.5)^2 + 10 x4 + 5 x5 + N(0, 1), so x6–x10 are irrelevant. The target is rescaled with (y + 3) / 36 and clipped to [0, 1] to fit a sigmoid output.

Columns: `x1`, `x2`, `x3`, `x4`, `x5`, `x6`, `x7`, `x8`, `x9`, `x10`, `y`. Target: `y`.

### regression/unscaled.csv

Regression with features on very different scales: tiny ~ N(0, 0.001), unit ~ U(0, 1), medium ~ N(50, 10), large ~ U(0, 1000), wide ~ N(0, 100) and skewed = exp(N(0, 1)). The target is a sigmoid of a weighted sum of the standardized features plus one interaction, with small noise. Training without feature scaling performs clearly worse.

Columns: `tiny`, `unit`, `medium`, `large`, `wide`, `skewed`, `y`. Target: `y`.

### regression/run-to-failure.csv

Bearings run until failure. Every machine has a random life of 800 to 2,000 operating hours and is measured every 10 hours. Wear grows slowly at first and accelerates towards the end (health = (hours / life)^2.5): BPFO amplitude, kurtosis and temperature rise. The target rul is the remaining useful life as a fraction of the total life (1 = new, 0 = failed). The machine column must be ignored; for a fair evaluation split by machine, not by row.

Columns: `machine`, `hours`, `rpm`, `temperature_c`, `rms_g`, `peak_g`, `crest_factor`, `kurtosis`, `amp_1x_g`, `amp_2x_g`, `amp_bpfo_g`, `rul`. Target: `rul`. Ignore: `machine`.

### regression/air-quality-calibration.csv

Calibration of a low-cost optical particle sensor (e.g. SDS011) against a reference instrument, one row per hour for a year. The true PM2.5 concentration follows the seasons (higher in winter), rush hours and occasional pollution episodes. The low-cost sensor has a gain error and overestimates strongly at high humidity (up to about five times) because particles absorb water; a cheap gas sensor (e.g. MQ-135) adds a correlated, temperature dependent signal. The target pm25_reference is the reference value divided by 150 µg/m³ and clipped to [0, 1]. Ignore timestamp.

Columns: `timestamp`, `hour`, `month`, `raw_pm25`, `temperature_c`, `humidity_pct`, `raw_no2`, `pm25_reference`. Target: `pm25_reference`. Ignore: `timestamp`.

### regression/smart-meter.csv

Hourly electricity consumption of a household with an electric heat pump, as read from a smart meter, for two years. Consumption has a base load, morning and evening peaks, more use on weekends, extra heating below 15 °C, random appliance runs and holidays away. Inputs are the calendar, the outdoor temperature and the consumption 24 and 168 hours earlier (forecasting the next hour). In about 1 % of the hours a fault or tampering multiplies or zeroes the reading; these are flagged in anomaly (1) for anomaly detection. The target load_kw is the consumption in kW divided by 8 and clipped to [0, 1]. Ignore timestamp and anomaly when forecasting.

Columns: `timestamp`, `hour`, `weekday`, `month`, `temperature_c`, `load_lag_24h_kw`, `load_lag_168h_kw`, `anomaly`, `load_kw`. Target: `load_kw`. Ignore: `timestamp`, `anomaly`.

## online

Streams for online learning: read the rows in order and feed them to online.Online one by one. The concept column (where present) tells which concept generated a row; ignore it for training and use it to evaluate drift detection.

| File | Task | Rows | Kind |
| --- | --- | --- | --- |
| [online/abrupt-drift.csv](online/abrupt-drift.csv) | classification | 100000 | benchmark |
| [online/gradual-drift.csv](online/gradual-drift.csv) | classification | 100000 | benchmark |
| [online/recurring-concepts.csv](online/recurring-concepts.csv) | classification | 100000 | benchmark |
| [online/incremental-drift.csv](online/incremental-drift.csv) | regression | 50000 | benchmark |
| [online/room-occupancy.csv](online/room-occupancy.csv) | classification | 16128 | sensor |
| [online/sensor-health.csv](online/sensor-health.csv) | classification | 60000 | sensor |
| [online/air-quality-aging.csv](online/air-quality-aging.csv) | regression | 17520 | sensor |

With gopher-learn:

```go
d, _ := synthetic.Lookup("online/abrupt-drift.csv")
set := learn.NewSet(nn.Classification)
_ = set.AddClass("A")
_ = set.AddClass("B")
o, _ := online.New(set, len(d.Features()), []int{8}, online.DefaultConfig())
i := 0
err := d.Read(".", func(r synthetic.Record) error {
	i++
	if err := o.Inject(learn.NewClassificationSample(r.Features, r.Target), true); err != nil {
		return err
	}
	if i%500 == 0 {
		_, _ = o.Iterate(ctx) // retrain regularly
	}
	return nil
})
```

### online/abrupt-drift.csv

A stream for online learning with an abrupt concept drift in the middle. x1–x4 are uniform in [-1, 1]; in the first half the label is A if x1 + x2 > 0, in the second half if x1 - x2 > 0. x3 and x4 are noise and 5 % of the labels are flipped. Read the rows in order; t is the time step and concept (1 or 2) the rule that generated the row. Ignore both for training.

Columns: `t`, `x1`, `x2`, `x3`, `x4`, `concept`, `label`. Target: `label`. Ignore: `t`, `concept`. Read in order.

### online/gradual-drift.csv

Like abrupt-drift, but the change is gradual: between 40 % and 60 % of the stream the probability that a row follows concept 2 (label A if x1 - x2 > 0) instead of concept 1 (A if x1 + x2 > 0) rises linearly from 0 to 1. x3 and x4 are noise, 5 % of the labels are flipped. Ignore t and concept for training.

Columns: `t`, `x1`, `x2`, `x3`, `x4`, `concept`, `label`. Target: `label`. Ignore: `t`, `concept`. Read in order.

### online/recurring-concepts.csv

Two concepts that take turns, like seasons: the stream switches between concept 1 (A if x1 + x2 > 0) and concept 2 (A if x1 - x2 > 0) every eighth of the stream. A learner that remembers old concepts recovers faster when they return. x3 and x4 are noise, 5 % of the labels are flipped. Ignore t and concept for training.

Columns: `t`, `x1`, `x2`, `x3`, `x4`, `concept`, `label`. Target: `label`. Ignore: `t`, `concept`. Read in order.

### online/incremental-drift.csv

Regression with a target that changes slowly over the whole stream: y = sigmoid(w(t) · x) with x1–x5 uniform in [-1, 1] and weights that move linearly from w = (2, -1, 1, 0, 0) to w = (-1, 1, 0, 2, -2). A model trained only on the beginning gets steadily worse; an online learner has to keep up. Small noise is added. Ignore t.

Columns: `t`, `x1`, `x2`, `x3`, `x4`, `x5`, `y`. Target: `y`. Ignore: `t`. Read in order.

### online/room-occupancy.csv

An office room measured every 5 minutes (e.g. BME280 for temperature, humidity and pressure, SCD30 for CO2, a light sensor and a PIR motion sensor), starting 2025-01-01. People come on working days roughly from 8:00 to 18:00 with a lunch break, sometimes stay late or come on a weekend. CO2 rises with every person and falls with ventilation and open windows, people warm the room and add humidity, lamps go on when someone is there, lamps are often left off on bright days, and the PIR sensor counts movements but misses people sitting still and has a few false triggers. The target occupied says whether anyone is in the room; people (the head count) can be used as an alternative regression target. Ignore timestamp and people when predicting occupied. The rows form a time series; read them in order for online learning.

Columns: `timestamp`, `hour`, `weekday`, `temperature_c`, `humidity_pct`, `pressure_hpa`, `co2_ppm`, `light_lux`, `motion_events`, `people`, `occupied`. Target: `occupied`. Ignore: `timestamp`, `people`. Read in order.

### online/sensor-health.csv

Health of an industrial temperature probe (e.g. PT100) in a process with setpoint changes, monitored together with a redundant second probe. Each row summarizes one minute of 1 Hz readings: mean, standard deviation, min, max, slope, the difference to the redundant probe and the number of missing samples. Faults occur as episodes of varying length: drift (slowly growing offset), stuck (frozen value), spikes (single outliers), dropout (missing samples) and noisy (electrical noise). About 75 % of the time the probe is ok. The rows form a stream; read them in order. Ignore t.

Columns: `t`, `mean_c`, `std_c`, `min_c`, `max_c`, `slope_c_per_min`, `diff_to_redundant_c`, `missing_samples`, `state`. Target: `state`. Ignore: `t`. Read in order.

### online/air-quality-aging.csv

A low-cost particle sensor (e.g. SDS011) running for two years next to a reference instrument, one row per hour. Dust in the optics makes the sensor read higher over time (its gain grows from 1.2 to 1.7), and after 16 months it is replaced by a new unit with a different gain and offset (sensor_unit changes from 1 to 2). Humidity still causes overestimation. An online calibration model must follow the slow drift and recover after the replacement. The target pm25_reference is the reference value divided by 150 µg/m³ and clipped to [0, 1]. Ignore timestamp and sensor_unit for training.

Columns: `timestamp`, `sensor_unit`, `hour`, `raw_pm25`, `temperature_c`, `humidity_pct`, `pm25_reference`. Target: `pm25_reference`. Ignore: `timestamp`, `sensor_unit`. Read in order.

## service

Event logs for the service package: every row is a prediction request whose feedback arrives feedback_delay seconds later, is sometimes wrong and is then corrected correction_delay seconds after it. Replay them in time order; the concept column marks the drift.

| File | Task | Rows | Kind |
| --- | --- | --- | --- |
| [service/offers.csv](service/offers.csv) | classification | 40000 | event log |
| [service/deliveries.csv](service/deliveries.csv) | regression | 40000 | event log |

With gopher-learn:

```go
// replay the log in time order: predictions at t, feedback at
// t+feedback_delay, corrections at t+feedback_delay+correction_delay
s, _ := service.New(net, nn.Classification, service.Config{Scaler: scaler, Reference: trusted})
p, _ := s.Predict(features)        // at t
_, err := s.Feedback(p.ID, feedback) // later; again with the true label as correction
// a complete replay: examples/online-service in gopher-learn
```

### service/offers.csv

Event log of an online shop that predicts whether a customer accepts an offer (accepted: yes or no). Features: price_ratio (own price / competitor price, 0.7–1.4), visits (previous visits, 0–20), basket_eur (value of the basket), hour (0–23) and discount_pct (0, 5, 10 or 20). Up to 60 % of the stream, customers mainly react to visits and the basket (concept 1); from then on they become price sensitive (concept 2). accepted is the true outcome. Its feedback arrives after feedback_delay seconds (median 10 minutes, empty for 5 % that never get feedback); 8 % of the feedback is wrong, and 70 % of the wrong ones are corrected correction_delay seconds later (median 1 day) with the true label. Replay the rows with the service package; t is in seconds.

Columns: `event`, `t`, `price_ratio`, `visits`, `basket_eur`, `hour`, `discount_pct`, `concept`, `accepted`, `feedback_delay`, `feedback`, `correction_delay`. Target: `accepted`. Ignore: `event`, `t`, `concept`, `feedback_delay`, `feedback`, `correction_delay`. Read in order.

### service/deliveries.csv

Event log of a delivery service that predicts the delivery time in minutes. Features: distance_km (0.5–25), items (1–30), traffic (0–1), hour (0–23) and rain (0 or 1). From 60 % of the stream on, a new depot makes deliveries about 20 % faster (concept 2). minutes is the true time, in raw units (not scaled to [0, 1]). The feedback arrives when the delivery is done (feedback_delay ≈ minutes·60 seconds; empty for 3 % without feedback). 1.5 % of the feedback is corrupt: reported in seconds instead of minutes, or with a typo (×10); 80 % of those are corrected correction_delay seconds later (median 2 hours). Replay the rows with the service package; t is in seconds.

Columns: `event`, `t`, `distance_km`, `items`, `traffic`, `hour`, `rain`, `concept`, `minutes`, `feedback_delay`, `feedback`, `correction_delay`. Target: `minutes`. Ignore: `event`, `t`, `concept`, `feedback_delay`, `feedback`, `correction_delay`. Read in order.

## text

Short texts for the string encoders (encoders.New with dictionary, split dictionary or n-grams).

| File | Task | Rows | Kind |
| --- | --- | --- | --- |
| [text/intents.phrase](text/intents.phrase) | classification | 10000 | benchmark |

With gopher-learn:

```go
enc := encoders.New("intents")
in := encoders.NewInput(encoders.String)
d, _ := synthetic.Lookup("text/intents.phrase")
_ = d.Read(".", func(r synthetic.Record) error { in.AddString(r.Text); return nil })
err := enc.Fit("text", in)
vector, _ := enc.EncodeString("text", "Wie wird das Wetter morgen?")
```

### text/intents.phrase

Short German utterances generated from templates for four intents (GREETING, WEATHER, ORDER, COMPLAINT), one "text#LABEL" per line like the ngram example. For the string encoders (dictionary, split dictionary, n-grams).

## loading

Edge cases for the loaders: delimiters, ignored columns, missing values and sparse LIBSVM files.

| File | Task | Rows | Kind |
| --- | --- | --- | --- |
| [loading/semicolon.csv](loading/semicolon.csv) | classification | 1000 | benchmark |
| [loading/missing-values.csv](loading/missing-values.csv) | classification | 1000 | benchmark |
| [loading/sparse.svm](loading/sparse.svm) | classification | 5000 | benchmark |

With gopher-learn:

```go
data := learn.NewSet(nn.Classification)
err := data.LoadCSV("loading/semicolon.csv",
	learn.WithComma(';'), learn.WithTarget("Art"), learn.WithIgnore("id"))
err = learn.NewSet(nn.Classification).LoadSVM("loading/sparse.svm")
// loading/missing-values.csv fails on purpose: "missing value" names the column
```

### loading/semicolon.csv

A CSV as exported by German spreadsheet tools: semicolon delimiter, an id column, column names with spaces and umlauts, and labels with spaces. Load it with learn.WithComma(';'), learn.WithTarget("Art") and learn.WithIgnore("id").

Columns: `id`, `Länge (cm)`, `Breite (cm)`, `Höhe (cm)`, `Art`. Target: `Art`. Ignore: `id`. Delimiter: `;`.

### loading/missing-values.csv

Like blobs but with 2 % empty fields. learn.Set.ReadCSV rejects it with a "missing value" error that names the column; it is the test case for a future imputation feature.

Columns: `x1`, `x2`, `x3`, `x4`, `label`. Target: `label`.

### loading/sparse.svm

Sparse LIBSVM data with 1,000 features and about 10 non-zero values per row. The label (+1/-1) is the sign of a weighted sum of the active features. Load it with learn.Set.LoadSVM.

## benchmark

Larger files for performance measurements.

| File | Task | Rows | Kind |
| --- | --- | --- | --- |
| [benchmark/large.csv](benchmark/large.csv) | classification | 250000 | benchmark |

With gopher-learn:

```go
data := learn.NewSet(nn.Classification)
err := data.LoadCSV("benchmark/large.csv", learn.WithTarget("label"))
```

### benchmark/large.csv

A larger binary classification set for performance measurements: 20 standard normal features, the label is drawn from a logistic model on x1–x8 (x9–x20 are noise).

Columns: `x1`, `x2`, `x3`, `x4`, `x5`, `x6`, `x7`, `x8`, `x9`, `x10`, `x11`, `x12`, `x13`, `x14`, `x15`, `x16`, `x17`, `x18`, `x19`, `x20`, `label`. Target: `label`.
