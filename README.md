# CRSP_ANcerno_match
Author: Shuang Chen (chenshuangjiayou@gmail.com).
This version: 2021 June 11.
The author welcomes all feedbacks.

# Purpose
This is to identify CRSP domestic equity active mutual funds in ANcerno through:
(1) comparing CRSP funds' change of holding between two timepoints and ANcerno accounts' cumulative trading between the same timepoints.
(2) comparing CRSP funds' fund name, fund management company name and other management team information with ANcerno accounts' manager reference table.

# Data
A. CRSP domestic equity active mutual funds are selected from CRSP Mutual Fund Database with the following conditions:
(1) crsp_fundno in CRSP can be matched with one unique wficn identifier through MFLINK1 table. (2) crsp_obj_cd starting with "ED" at least once in its lifetime recorded in CRSP.
(3) the sum of common shares, preferred shares and other equity shares is over 80% on average over its lifetime.
(4) not an index fund or fund of funds or ETF or ETN.

B. CRSP domestic equity active mutual funds' holdings are collected from two sources: (1) from 2010 July 1 to 2014 December 31, holdings data are from CRSP due to its high quality and mostly monthly report frequency. CRSP holdings data use crsp_portno as identifier. To convert the identifier to wficn, the observations are deleted when one wficn corresponds to multiple crsp_fundnos belonging to different crsp_portnos. (2) before 2010 July 1, holdings data are from Thomson Reuters S12.

C. Only keep the stocks with shrcd in 10,11,12,18 (ordinary common shares with no special status found, no special status necessary, foreign incorporated or REIT) in both CRSP funds' holding data and ANcerno accounts' trading data. Multiply the numbers of shares in CRSP funds' holdings with CRSP monthly cfacshr multiplier. Multiply the numbers of shares in ANcerno trading orders with CRSP daily cfacshr multiplier. The adjusted numbers of shares take stock split, stock dividends etc. into account and are comparable over time.

D. The ANcerno accounts' manager reference table is a historical snapshot on 2011 Sep 6. All time periods of ANcerno data are with the same historical manager information. To be comparable with ANcerno, for periods after 2011 Sep 6, pick the earliest fund name and management team information for each CRSP fund. For periods before 2011 Sep 6, pick the latest one for each CRSP fund. Then if one CRSP fund has 2 records, keep the earlier one for this fund's whole lifetime.

# Methodology
1. Select qualified CRSP funds and time periods.
Compare one CRSP fund's holdings in adjacent report dates. This CRSP fund's change of holdings between two adjacent report dates will enter the comparison with ANcerno cumulative tradings if:
(1) at least 10 stocks change their adjusted numbers of shares,
(2) the interval between the two adjacent report dates is at most 6 months.

2. Calculate 3 types of distance between CRSP change of holdings and ANcerno cumulative tradings.

(1) create CRSP change of holdings vector and ANcerno cumulative tradings vector. Each dimension of these two vectors is a stock with CRSP change of holdings != 0. Each element in CRSP change of holdings vector is the change of shares on this stock held by a fund. Each element in ANcerno cumulative tradings vector is the sum of shares traded on this stock by an account.

(2) distance 1: cosine similarity between the two vectors.
distance 2: euclidean distances between the two vectors, then divided by the length of CRSP change of holdings vector. distance 3: divide each element in ANcerno cumulative tradings vector by the corresponding element in CRSP change of holdings, which generates a quotient vector. Then calculate the average value of all elements in this quotient vector.

(3) only keep the CRSP-ANcerno pair with distance 3 in [0.001,1000], which guarantees the ANcerno account is not a trivial part of the CRSP fund and vice versa. At the same time, to be holding-matched, the CRSP-ANcerno pair has to satisfy one of the following two conditions: distance 1 >= 0.9 or distance 2 <= 0.1.

3. Compare fund name and fund management company.
The available fund name and fund management company information for CRSP funds includes: (1) CRSP Mutual Fund database: fund_name, mgmt_name, mgr_name, adv_name. (2) MFLINK2 & Thomson Reuters S12: fundname, mgrco. (3) Management Team Section in fund summary prospectus in EDGAR.

The available fund name and fund management company information or ANcerno accounts is manager and reportedmanager in manager reference table.

Do basic text cleaning on these texts, such as removing common words in fund name or fund management company. E.g. "Corp", "Investments", "Funds".

Obtain fuzzy match scores between mgmt_name & manager, adv_name & manager, mgrco & manager, fund_name & reportedmanager, fundname & reportedmanager, mgr_name & reportedmanager.

If any fuzzy match score is above 70 or manager/reportedmanager appears in Management Team Section in fund summary prospectus, then consider the CRSP-ANcerno pair as name-matched.

4. Assign priority score to matched CRSP-ANcerno pairs
(Highest)
Priority 0: distance 2 <= 0.1 and name-matched. Once these two conditions are satisfied, distance 1 >= 0.9 is also satisfied.

Priority 1: (distance 2 <= 0.1 but not name-matched) or (distance 1 >= 0.9 and name-matched but not in Priority 0)

Priority 2: CRSP-ANcerno pairs with the same CRSP identifier and ANcerno identifier of pairs in Priority 0 and Priority 1 but not satisfying Priority 0 or Priority 1 during these periods.

Priority 3: not matched above, but with distance 1 >= 0.95 and its ANcerno manager and reportedmanager are missing.
(Lowest)

# Results
Distribution of identified CRSP funds in ANcerno by month and by matching priority
