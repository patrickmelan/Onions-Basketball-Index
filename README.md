# NCAA March Madness Bracket Predictor

A probabilistic bracket prediction model for the 2025 NCAA Men's Basketball Tournament, built on KenPom efficiency data.

## Data

All stats come from [KenPom](https://kenpom.com/) ratings through March 15, 2026.

- `kenpom25.csv` — Raw KenPom data for all D1 teams (21 columns including stat rankings)
- `kenpom25_CLEAN.csv` — Cleaned dataset with only the 68 tournament teams and 16 features
- `kenpom.ipynb` — All analysis, models, and bracket predictions

### Features Used

| Feature | Description |
|---------|-------------|
| NetRtg | Net efficiency rating (points per 100 possessions margin) |
| ORtg | Adjusted offensive efficiency |
| DRtg | Adjusted defensive efficiency (lower is better) |
| AdjT | Adjusted tempo (possessions per 40 minutes) |
| Luck | Deviation between actual wins and expected wins |
| NetRtgSOS | Strength of schedule based on net efficiency |
| ORtgSOS / DRtgSOS | Offensive and defensive strength of schedule |
| NetRtgNCSOS | Non-conference strength of schedule |

## Models

### v1 — NetRtg Logistic Model

Baseline model. Uses each team's net efficiency rating and a logistic function to compute win probability:

```
win_prob = 1 / (1 + 10^(-diff / 15.0))
```

Where `diff` is the NetRtg gap between the two teams and `15.0` is a scaling factor (a 15-point NetRtg difference gives roughly a 75% win probability). Runs 100,000 Monte Carlo simulations to produce championship probabilities for every team.

### v2 — Balance-Adjusted Variance

Same expected value as v1, but uses ORtg and DRtg to model game **variance**. Each team gets a "balance" score measuring how lopsided they are (all offense vs. all defense). Lopsided matchups get a wider scaling factor, pushing win probabilities closer to 50/50 — reflecting the real-world volatility of one-dimensional teams.

A mathematically important note: the "correct" matchup-based approach (team A's offense vs. team B's defense) actually reduces to plain NetRtg comparison because KenPom's ratings are already adjusted against average opponents. The real value of splitting ORtg/DRtg is in modeling uncertainty, not expected margin.

### v3 — Upset Predictor

Builds on v2 and adds an upset detection layer. Each matchup gets an **upset score** based on five signals:

| Signal | Weight | Reasoning |
|--------|--------|-----------|
| Close win probability | 40 | Tight games produce more upsets |
| Favorite's Luck stat | 20 | Lucky teams tend to regress in March |
| Underdog's Luck stat | 15 | Unlucky teams are due for a bounce back |
| Favorite's lopsidedness | 0.5 | One-dimensional teams are volatile |
| Underdog's SOS advantage | 0.3 | Battle-tested underdogs overperform |

The model picks upsets when the score exceeds a round-dependent threshold (18 in R64, 21 in R32, 24 in S16, etc.) and the underdog has at least a 30% win probability. The threshold increases each round because upsets become harder to predict deeper in the tournament.

Favorites are determined by **seed**, not KenPom ranking — so a 5-seed beating a 4-seed counts as an upset even if the 5-seed has a higher net rating.

## Results

The v3 model picks Duke as the 2025 national champion, beating Arizona in the final. Monte Carlo simulations (100k iterations) give the following championship probabilities:

| Team | Champion % |
|------|-----------|
| Duke | 33.2% |
| Arizona | 22.5% |
| Michigan | 20.0% |
| Florida | 6.4% |
| Houston | 4.8% |
| Iowa St. | 3.7% |
| Illinois | 3.0% |
| Purdue | 2.6% |

## Running

Open `kenpom.ipynb` in Jupyter and run the cells top to bottom. The notebook is self-contained — it reads the raw CSV and produces cleaned data, bracket predictions, and simulation results.

Requires Python 3 with `pandas`.
