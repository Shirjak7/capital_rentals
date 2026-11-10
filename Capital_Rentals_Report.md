# Capital Rentals: Data Cleaning and Findings

MATH 153 · Data Wrangling and Data Engineering · Group Project
Team members: Karma, Melvin, Risav, Shirjak

## Overview

Our team worked with Washington, DC rental data collected for a tenant-advocacy nonprofit. The raw data covered July through October 2026 and included two listing spreadsheets, a landlord registry, 32 text advertisements, neighborhood information, and a monthly rent-price index. Listing details included rent, bedrooms, bathrooms, square footage, and posting dates. The main challenges were inconsistent formats, missing square footage, repeated listings, and suspicious values.

## 1. Combining and cleaning the sources (Parts 1 and 2)

In the first sections, we combined and organized these sources. We standardized column names, converted rent into numeric values, and formatted dates consistently. Combining 180 Q3 records with 126 Q4 records produced 306 rows.

Six listing IDs were repeated across Q3 and Q4. Each pair was identical except for its source label, so we kept the Q4 copy. Removing the extra copies prevented double-counting and left 300 listings.

We standardized 59 neighborhood-code spellings into 12 codes and used the advertisements to fill all 32 missing square-footage values.

## 2. Joining the data (Part 3)

Parts 3 and 4 focused on preparing, integrating, and reshaping data to support accurate housing market analysis and nonprofit decision-making.

In Part 3, we transformed nested landlord data into a usable tabular format, created a new variable measuring landlord licensing activity, and merged multiple datasets to build a comprehensive master dataset containing approximately 300 listings and 25 variables. The outer merge of listings and landlords showed a data consistency rate of about 98.7%: 297 of the 301 rows in the merge matched. The few unmatched records were three listings from landlord LL199, who is not in the registry, and one registered landlord, LL125, with no listings. Both groups highlight potential data quality issues requiring further verification.

We also compared merge strategies and determined that a left join was the most appropriate approach, because it preserved the full housing inventory (300 listings, compared with 297 under an inner join) while minimizing data loss. Additionally, we incorporated neighborhood-level information and used merge validation (`validate="many_to_one"`) so that a duplicated neighborhood key could not silently multiply listing rows.

## 3. Reshaping and rent trends (Part 4)

In Part 4, we reshaped the rent-price index between wide and long formats to facilitate analysis and calculated each neighborhood's change from January to October. Rental trends varied substantially: Petworth had the largest increase (+7.51%) and Anacostia the largest decrease (-5.26%).

We also produced pivot table summaries of median rent by neighborhood and bedroom count, which revealed significant affordability differences across the city, including Georgetown's substantially higher rental prices compared with lower-cost neighborhoods such as Anacostia.

Together, these tasks improved data quality, expanded analytical capabilities, and generated insights into housing inventory, rental growth patterns, and neighborhood affordability.

## 4. Validity checks and outliers (Part 5)

This section covers evaluating outliers and preparing the cleaned data for analysis. First, we checked basic validity rules, including positive rent and square footage. One listing had zero square feet and was excluded before calculating monthly rent per square foot.

On the remaining 299 records, we applied three outlier rules to both rent and rent per square foot: absolute z-scores above 3, absolute modified z-scores above 3.5, and values outside the 1.5-times-IQR fences. The \$665,000 monthly rent inflated the mean and standard deviation, masking other high rents under the ordinary z-score rule. Modified z-scores and IQR checks were less affected by that extreme value.

Our removal decisions considered the listing details alongside the statistical flags. We excluded the zero-area listing and three likely entry errors: \$665,000 monthly rent, an 85-square-foot two-bedroom apartment, and \$27.50 monthly rent for a one-bedroom apartment. Because no source confirmed these values, we preserved the original records and documented the exclusions. We retained the \$14,800 Georgetown and \$9,900 Navy Yard listings as plausible luxury rentals based on their size, bedrooms, bathrooms, and location.

After deduplication and the four exclusions, the cleaned dataset contained 296 listings.

## 5. Scaling and transformations (Part 6)

On the cleaned dataset of 296 listings, we applied three transformations to rent and square footage: min-max scaling, standardization, and log(1 + x). We verified that the min-max columns run from exactly 0 to 1 and that the standardized columns have a mean of essentially zero and a standard deviation of 1.

Rent is right-skewed, with a skewness of 1.31. After the log(1 + x) transformation the skewness was -0.33, much closer to symmetric. Min-max scaling and standardization only rescale the values linearly, so they leave the shape of the distribution unchanged. For a right-skewed price, log(1 + x) is therefore the better choice.

The order of the steps matters. When we applied min-max scaling to the uncleaned rent, the \$665,000 entry became the maximum, and even the largest retained rent (\$14,800) mapped to only 0.0222. A median listing scaled to about 0.005 on the uncleaned range, compared with about 0.18 on the cleaned range. Scaling depends on the minimum and maximum, which outliers distort, so the data should be cleaned first and scaled second.

We use log(1 + x) rather than log(x) because log(0) is undefined, while log(1 + 0) is 0. In this project, the bedroom column contains zeros for studios, so log(1 + x) would keep those listings finite.

## 6. SQL database (Part 7)

We stored the cleaned listings, 25 registered landlords, and 12 neighborhoods in a SQLite database (`rentals.db`). Queries found 56 listings above \$5,000 and 38 below \$1,500. Of 18 two-bedroom listings below \$2,500, 17 were in Anacostia or Congress Heights. Three listings belonged to a landlord missing from the registry and remained flagged for follow-up.

## 7. Summary of findings

The cleaned data had a mean monthly rent of **\$3,462.43** and a median of **\$3,290**. The median better represents a typical listing because high rents still pull the mean upward. Creating rent per square foot was especially useful: it exposed suspicious records that rent alone missed and helped distinguish likely errors from plausibly expensive housing.

These findings describe the project's synthetic advertised rents rather than verified payments by DC renters.
