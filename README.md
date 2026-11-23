# CFDE Data Quality Assessment

By the Sustainability Core at ICC

The CFDE Data Quality Assessment reviews the publicly available data of NIH Common Fund (CF) Programs, as held in each CF Program's own web portal, and reports on three aspects of data quality:

- Completeness, which identifies missing values such as fields left blank, and entries such as "NA", "null" or "-999";
- De-duplication, which detects duplicate records that appear more than once, and identifiers (e.g., subject or sample IDs) that are assigned to more than one record;
- Conformance, which uncovers format errors such as entries that are not numbers in fields meant to hold numbers, a single term recorded in several forms (for example "Female", "female" and "FEMALE"), stray spaces before or after a value, and values that do not follow the required format or are not among the accepted terms.

The assessment does not identify outliers in the data. For programs whose data are access-controlled, the assessment reviews the publicly available metadata or the open-sourced data subsection.

## Install

You need Python 3.9 or newer.

```
git clone https://github.com/cfde-sustainability/CFDE-Data-Quality-Assessment.git
cd CFDE-Data-Quality-Assessment
pip install -r requirements.txt
```

MoTrPAC publishes its tables as R data files, which need one more package:

```
pip install pyreadr
```

On Python 3.9 this package sometimes fails to install. Everything except the
MoTrPAC check works without it.

## Check your own file

This is the quickest way to try it. Give it a CSV or TSV file and tell it which
columns are required and which must be unique:

```
python3 -m qualitycheck file my_table.csv --required subject_id,sex,age --ids subject_id
```

Other options:

| Option     | What it does                                                    |
|------------|-----------------------------------------------------------------|
| --key A,B  | columns that together must be unique (for example dataset and subject_id) |
| --why TEXT | where your list of required fields comes from                   |
| --sep      | the column separator, if it is not clear from the file name     |
| --limits   | your own limits file (see Limits below)                         |
| --json     | also save the results to a JSON file                            |

There is a small real file to try it on in tests/data (the first 40 rows of the
public GTEx v11 subject table):

```
python3 -m qualitycheck file tests/data/gtex_subjects_first40.txt --required SUBJID,SEX,AGE --ids SUBJID
```

## Check the repositories

See which repositories are covered:

```
python3 -m qualitycheck list
```

Run one or a few:

```
python3 -m qualitycheck run gtex hmp
```

Or all of them:

```
python3 -m qualitycheck run all
```

Run these from the CFDE-Data-Quality-Assessment folder, because `data/` and `results/`
are made in the current folder. Downloads are saved in `data/` so a second run is much faster. Results go to
`results/<repository>.json`. If one site is down, the others still run and the
missing one is listed at the end.

Some runs are big. Kids First reads a C2M2 package with over a million file
records, SPARC reads every subjects and samples file in about 340 datasets, and
GTEx reads four full read count matrices. Expect these to take a while and to
use a few GB of disk.

## Make the Excel report

```
python3 -m qualitycheck report
```

This reads everything in `results/` and writes `CFDE-Data-Quality-Assessment.xlsx`
with these sheets:

| Sheet          | What is in it                                                |
|----------------|--------------------------------------------------------------|
| Summary        | one row per repository, with the result for each area        |
| Missing values | required and optional missing counts for every table          |
| Checks         | every check: records checked, problems found, result          |
| Columns        | every column: empty cells, hidden missing, unique values, one example value |
| Method         | the steps, the limits and the missing values score            |
| Sources        | where each table came from, and the standards used            |

## Reading a result

Each check gets a problem % (problems found divided by records checked), and
the limits turn that into a result:

- Pass: at or below the pass limit
- Check: between the two limits, worth a look
- Fail: above the fail limit
- No records: there was nothing to check

On the Summary sheet each area shows its worst result.

## Limits

The limits are in `qualitycheck/limits.json`. They are working numbers for this
review, not a published standard, so change them if your team agrees on others.
In the Excel file the same limits are the yellow cells on the Method sheet, and
changing one there updates every Result cell without running the code again
(the short findings text on the Summary sheet does not change).

The missing values score weights required fields 0.8 and optional fields 0.2.
Those are in the same file.

## Partial reads

A few repositories are too large to read whole, so only part is checked and the
table name says so:

- 4DN BED files: 10 files picked at random with a fixed seed
- LINCS: the first 20,000 signatures of the L1000 chemical library, sorted by ID
- Metabolomics Workbench: the 7 studies that have data, out of 11

## Tests

```
pip install pytest
python3 -m pytest tests
```

## Things to know

- Repositories change. A run next month can give different counts from this
  month, because data is added or fixed. The Sources sheet has every link used.
- If a download link moves, the run for that repository stops with the error.
  The link is near the top of its file in `qualitycheck/repos/`.
- NPH has no public data yet, so it shows as Not tested.

## Contact

For questions about this assessment, contact the Sustainability Core at
[cfde.sustainability@gmail.com](mailto:cfde.sustainability@gmail.com).

## License

MIT, see [LICENSE](LICENSE).
