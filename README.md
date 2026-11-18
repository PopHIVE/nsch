# NSCH

National Survey of Children's Health (NSCH): single-year, design-based state and
national estimates for [PopHIVE](https://pophive.org), computed from the U.S.
Census Bureau public-use topical files (2016 onward), sponsored by HRSA MCHB.

PopHIVE/Ingest mirrors `standard/*.csv.gz` and `measure_info.json` from this
repository into `data/nsch`, where they feed the youth wellbeing bundle.

## Method

- **Raw:** `raw/nsch_{year}_extract.csv.gz` holds only the variables `ingest.R`
  uses, with Census missing-value codes kept as `.M` / `.L` / `.N`. The full Stata
  zips are cached in `raw/zip_cache/` (gitignored) and downloaded again only when a
  file's Last-Modified / size on www2.census.gov changes.
- **Measure coding:** an R port of the Census/HRSA Title V measure code
  (`Title _V_Code_SAS_2016-2025.sas`); each measure records its Title V variable in
  `measure_info.json`.
- **Estimation:** weights `FWC`, strata `FIPSST x STRATUM`, PSU `HHID`, domain
  estimation, 95% logit CIs. Following the NSCH Comprehensive Analytic Guide, cells
  with an unweighted denominator under 30 are suppressed and CIs wider than 20
  points or 1.2x the estimate are flagged `_unreliable`.

Flourishing: `nsch_pct_flourishing_6_17` / `_6mo_5` are "meets all items" (the
figure published by the DRC at childhealthdata.org; reproduced exactly for
2022-23). The Title V National Outcome Measures (`FlrshChild` / `FlrshYoung`),
which judge a child only on the items answered and run slightly higher, are
deliberately not reported.

Most measures are Title V definitions (`title_v_variable` in measure_info). A few
are a single NSCH question reported directly (`nsch_variable`), e.g.
`nsch_pct_share_ideas_well_6_17` (K8Q21 = "Very well" or "Somewhat well").

## Output

`standard/data_age.csv.gz`, `data_age_sex.csv.gz` and
`data_age_race_ethnicity.csv.gz`, wide, keyed on `geography` (`"00"` national,
2-digit state FIPS) + `time` (`YYYY-12-31`, survey year) + `age` (`0-5`, `6-11`,
`12-17`, `Overall` = 0-17) and the stratifier. Each measure has `_lcl`, `_ucl`,
`_n`, `_suppressed`, `_unreliable` and `_not_asked` columns; suppressed and
not-asked estimates are left missing.

## Updating

Census releases a new survey year each autumn. From the repository root:

```r
source("ingest.R")
```

It checks www2.census.gov, downloads only new or changed years, and rebuilds the
standard files when anything changed. `process.json` records the files and
variables the extracts were built from. Adding a variable to `KEEP_VARS` rebuilds
every year's extract on the next run; adding or removing a measure rebuilds the
standard files.

`measure_info.json` is regenerated on every run (except `_catalog`, which is
preserved), so edit descriptions in `ingest.R`, not in the JSON. Commit and push
the updated `standard/`, `raw/`, `measure_info.json` and `process.json`; the
Ingest mirror picks them up on its next daily build.

`resources/all_fips.csv.gz` is a copy of the PopHIVE/Ingest FIPS crosswalk.
