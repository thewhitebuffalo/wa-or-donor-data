# Washington & Oregon campaign donor, PAC, and lobbyist data

Pulled **2026-10-16** from official state sources. All files are gzipped CSV (UTF-8, header row). Every file is under 100 MB.
These are **public records** published by Washington and Oregon state agencies. They have not been edited except for format conversion (Oregon's XLSX exports were converted to CSV) and, for the Oregon contributions, merging the chunked exports with duplicate transaction IDs removed (none were found). Filer-reported values, including typos, are kept as-is.

## Reassembling split files
Large files are split into `.partNN.csv.gz`. Each part has its own header row.
```bash
# Combine into a single CSV, keeping the header only once
first=1; for f in "washington/WA Donor Contributions FULL - 2026-10-16".part*.csv.gz; do
  if [ $first = 1 ]; then zcat "$f"; first=0; else zcat "$f" | tail -n +2; fi
done > "WA Donor Contributions FULL.csv"
```
pandas: `pd.concat(pd.read_csv(f) for f in sorted(glob.glob('washington/WA Donor Contributions FULL*.gz')))`

## washington/  (source: Washington Public Disclosure Commission, published on data.wa.gov)
| File | Dataset ID | Rows | Date range |
|---|---|---|---|
| WA Donor Contributions FULL (11 parts) | kv7h-kjye Contributions to Candidates and Political Committees | 6,397,632 | all years (receipt_date 0202 to 2041, see gaps) |
| WA Donor Contributions 2024-2026 (3 parts) | kv7h-kjye, filtered to election_year >= 2024 | 917,202 | election years 2024+ |
| WA Contributions to out-of-state political committees | qdtg-6yir | 612 | election years 2023–2026 |
| WA Expenditures by out-of-state political committees | mzg4-pm9n | 14,362 | election years 2007–2026 |
| WA Independent Campaign Expenditures and Electioneering Communications | 67cp-h962 | 40,295 | report_date 2009-01-21 to 2026-10-16 |
| WA Campaign Finance Summary (candidates/PACs/committees) | 3h9x-7bvm | 57,820 | receipt_date 2007 to 2026-10-15 |
| WA Lobbyist Compensation and Expenses by Source | 9nnw-c693 | 338,365 | receipt_date 2014-05-09 to 2026-10-16 |
| WA Lobbyist Employment Registrations | xhn7-64im | 28,250 | employment_year 2016–2028 |
| WA Lobbyist Summary | c4ag-3cmj | 10,325 | year 2016–2028 |
| WA Lobbyist Employers Summary | biux-xiwe | 16,726 | year 2016–2028 |

**kv7h-kjye columns (38):** id, report_number, origin, committee_id, fund_id, filer_id, type (Candidate / Political Committee), filer_name, office, legislative_district, position, party, ballot_number, for_or_against, jurisdiction, jurisdiction_county, jurisdiction_type, election_year, amount, cash_or_in_kind, receipt_date, description, memo, primary_general, code, contributor_category, contributor_name, contributor_address, contributor_city, contributor_state, contributor_zip, contributor_occupation, contributor_employer_name, contributor_employer_city, contributor_employer_state, url, contributor_location, row_checksum.
Column lists for the other datasets are in each CSV's header row and on `https://data.wa.gov/d/<dataset id>`.

## oregon/
| File | Source | Rows | Date range |
|---|---|---|---|
| OR Donor Contributions 2024-2026 | Oregon Secretary of State ORESTAR transaction search (Transaction Type = Contribution, all filers incl. PACs) | 385,729 | 2024-01-01 to 2026-10-16 |
| OR ORESTAR Committees (incl PACs, active and discontinued) | ORESTAR committee search | 11,385 | filing dates 1984 to 2026-10-16 |
| OR Lobbyist Registrations | Oregon Government Ethics Commission (OGEC) EFS Public Records | 24,650 | registration periods 2014-2015 to 2026-2027 |
| OR Lobbyist Client-Employer Registrations | OGEC EFS | 23,420 | same |
| OR Lobbyist Expenditures (quarterly) | OGEC EFS | 10,168 | years 2015–2026 |
| OR Lobbyist Client-Employer Expenditures (quarterly) | OGEC EFS | 12,253 | years 2015–2026 |

**ORESTAR contribution columns (45):** Tran Id, Original Id, Tran Date, Tran Status, Filer (recipient committee), Contributor/Payee, Sub Type, Payer of Personal Expenditure, Amount, Aggregate Amount, Contributor/Payee Committee ID, Filer Id, Attest By Name, Attest Date, Review By Name, Review Date, Due Date, Occptn Ltr Date, Pymt Sched Txt, Purp Desc, Intrst Rate, Check Nbr, Tran Stsfd Ind, Filed By Name, Filed Date, Addr book Agent Name, Book Type, Title Txt, Occptn Txt, Emp Name, Emp City, Emp State, Employ Ind, Self Employ Ind, Addr Line1, Addr Line2, City, State, Zip, Zip Plus Four, County, Country, Foreign Postal Code, Purpose Codes, Exp Date.
OGEC files hold totals by quarter (lobbyist: Year, name, Q1–Q4, Total; client: also Business Name). Registration files list period, lobbyist/client names, status, and registration date.

## Known gaps and caveats
- **WA dates are messy:** filers sometimes type dates wrong (receipt_date values as early as year 0202 and as late as 2041; similar typos in the out-of-state and summary sets). Use election_year where possible.
- **OR contributions cover only 2024-01-01 to 2026-10-16.** Earlier years can be pulled the same way but were not included.
- **ORESTAR search limits:** each search/export returns at most 5,000 records, so the data was pulled in date and amount slices (and, for 2024-12-23, by the first letter of the contributor name, because that day had 5,661 contributions of exactly $15.63). Recovered counts matched the counts ORESTAR reported. One weekly slice (2024-05-06 to 05-19, 4,946 records) was close to the cap. It was taken as complete but was not cross-checked, so it could be slightly short.
- ORESTAR's committee list comes from the same search interface, which has the same result limits. Committees were not checked against any separate master list.
- OGEC lobbying data is quarterly totals, not itemized transactions.
- The WA 2024+ file is a subset of the WA FULL file (they overlap).
