# sml-salesinsights-common

Shared SML package with the common datasets, dimensions, and row security for the Sales Insights demo catalogs.

This repository is an SML package. An SML package is a Git repository with reusable SML (Semantic Modeling Language) objects that other SML repositories import through their `package.yml`. It is not a Git submodule. The three Sales Insights catalogs import it, so the customer, date, geography, gender, and product dimensions are defined once:

- [sml-salesinsights-databricks](https://github.com/muellermarco/sml-salesinsights-databricks)
- [sml-salesinsights-gbq](https://github.com/muellermarco/sml-salesinsights-gbq)
- [sml-salesinsights-snowflake](https://github.com/muellermarco/sml-salesinsights-snowflake)

## Technologies

- AtScale 2026.9.2
- sml-cli 2025.12.0 (pinned in the [atscale-cicd-pipeline](https://github.com/muellermarco/atscale-cicd-pipeline), which runs `sml-cli install` before validation)
- SML `version: 1` (from the `package.yml` of the consumer repositories)
- Databricks, Google BigQuery, and Snowflake (one branch each)

## Branches: one per data platform

Each data platform has its own branch. The branches share the same object names, but each branch has its own connection files with `override: true`, which point the objects to the tables on that platform.

| Branch | Connection files | Imported by |
|---|---|---|
| `main` | `connections/security_connection.yml` | none |
| `databricks` | `connections/common_connection.yml`, `connections/security_connection.yml` | sml-salesinsights-databricks |
| `gbq` | `connections/security_connection.yml` | sml-salesinsights-gbq |
| `snowflake` | `connections/common_connection.yml`, `connections/security_connection.yml` | sml-salesinsights-snowflake |

The `databricks` and `snowflake` branches contain more objects than `main`: an inventory fact dataset, the `Stockbalance` metric, an `Inventory` model, extra product dimensions (for example `Color Dimension` and `Size Dimension`), and more row security objects.

## Contents of the main branch

| Folder | Objects |
|---|---|
| `datasets/` | 9 datasets: `SECURITY COUNTRY USER`, `datecustom`, `dim_geo_city`, `dim_geo_country`, `dim_geo_postalcode`, `dim_geo_state`, `dimcustomer`, `dimgender`, `dimproduct` |
| `dimensions/` | 5 dimensions: `Customer Dimension`, `Date Dimension`, `Gender Dimension`, `Geography Dimension`, `Product Dimension` |
| `security/` | 1 row security object: `Security Dim Country by User` (filters the country by user ID) |
| `connections/` | `security_connection` |

## How a catalog imports the package

Each consumer repository has a `package.yml` at its root. It names the package `Common-Objects`, the branch for its platform, and pins one commit:

```yaml
version: 1
packages:
  - name: Common-Objects
    url: https://github.com/muellermarco/sml-salesinsights-common.git
    branch: snowflake
    version: commit:<sha>
```

Because the version is a commit, a change in this repository has no effect on a catalog until that catalog's `package.yml` points to the new commit.

## Change a shared object

1. Create a feature branch from the platform branch, for example `snowflake`.
2. Edit the SML files.
3. Open a pull request to the platform branch and merge it.
4. Copy the full SHA of the merge commit: `git rev-parse origin/snowflake`.
5. Open the consumer repository for that platform.
6. In `package.yml`, set `version: commit:<sha>` to the new SHA.
7. Run `npx -y sml-cli@2025.12.0 install .` and then `npx -y sml-cli@2025.12.0 validate .`.
   Result: the output contains `Validation SUCCESSFUL`.
8. Open a pull request in the consumer repository.
   Result: the `validate` check is green.
9. If the change applies to more than one platform, do steps 1 to 8 for each platform branch.

## License

No `LICENSE` file is in this repository. The consumer repositories use the MIT License.
