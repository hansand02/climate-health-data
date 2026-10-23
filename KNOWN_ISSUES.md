# Known issues

Problems found while adding the chap-core and GitHub datasets (October 2026). The data files are
left as they came from their source. Each dataset's schema has the details in its `notes`.

## Climate on the wrong locations (13 ISIMIP files)

`rainfall` and `mean_temperature` belong to other locations than the row says in:
argentina, brazil_2001-2017, colombia, ecuador, el_salvador, indonesia, malaysia, mexico,
nicaragua_2000-2005, paraguay, peru, thailand_2000-2017, vietnam_2000-2017 (all under `dengue/`).
Cases and population are joined by name and look fine. Cambodia, Laos and Panama (10 or fewer
locations) look fine.

Evidence:

- January mean temperature in `vietnam_2000-2017`: HoChiMinh 15.5 °C, HaNoi 21.5 °C (the
  south is colder than the north). In `thailand_2000-2017`: Narathiwat (far south) 19.9 °C,
  Chiang Rai (far north) 23.4 °C.
- Thailand: for all 27 provinces that can be compared with `thailand_1993-2022`, the series
  matches another province (median RMSE 0.14 °C) better than its own (1.80 °C).
- Vietnam: 15 of 19 series are exact copies of other provinces' series in
  `vietnam_1994-2010` (same chap-core pipeline).
- Brazil: 18 of 21 states match another state better than their own against Lowe et al. (CRU).
- The other 10 files: same build code and physically implausible values (e.g. Mexico, Tabasco
  8.5 °C in January), but no independent reference to compare with.

Probable cause: chap-core `scripts/full_dataset.py` at commit f02e58952, line 89,
`zip(names, data.values())`. The climate comes back grouped by feature id as text ("0", "1",
"10", "11", ..., "2"), so names and series are paired in different orders. This predicts the
Thailand and Vietnam mismatches exactly, and explains why files with 10 or fewer locations are
unaffected. Not confirmed by rerunning the build.

## Population

- `dengue/nicaragua_2003-2022`: random placeholder, `100000 + randint(1000)` (chap-core
  `scripts/open_dengue/nicaragua.py`).
- `malaria/madagascar_2016-2022`: all 1 (placeholder in the source model).
- `dengue/vietnam_1994-2010`: QUANG NINH 1,437 and KHANH HOA 2,922 (real ~1.2 million each);
  PHU THO is 42% of the value in `vietnam_1998-2010`. Cause unknown.
- `malaria/rwanda_2015-2024`: districts sum to 24-28 million (2015-2020) and ~17 million
  (2021-2024); Rwanda has ~13 million people. Definition unknown.

## Other

- `dengue/nicaragua_2003-2022`: weather is from admin-2 municipalities that share a name with
  the department (the script asks for level 2 polygons), only 10 departments kept. Temperature in
  K, rainfall in m (mean of daily values).
- `dengue/brazil_2000-2019`: rainfall is 0 in every row (the source, Lowe et al., has no
  rainfall). Unusable as a climate dataset.
- `dengue/laos_2023-2024`: cases only. The empty weeks are probably zero cases, since DHIS2
  analytics omits zeros.
- `dengue/laos_1998-2010`, `thailand_1993-2022`, `vietnam_1998-2010`: rainfall is mm/day
  (mean daily), not a monthly total. The chap-core files are mm/month.
- `malaria/rwanda_2015-2024`: origin undocumented (found in talk slides).
