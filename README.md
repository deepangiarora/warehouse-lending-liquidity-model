# Warehouse Lending & Liquidity Model

I built this model to understand how an auto lender actually funds its loans before they get securitized, and what happens to its cash when something goes wrong. It is a hypothetical non-prime auto lender with a $40M bank warehouse line, but I based the structure on a real 2025 auto warehouse agreement filed with the SEC, and the pricing and credit numbers on real 2025 and 2026 data.

Everything is in Excel with live formulas, so if you change anything on the Inputs sheet, the whole model updates.

## I. How warehouse lending works

When an auto lender makes a car loan, it has to pay the dealer right away, but it usually does not have enough cash to hold thousands of loans on its own. So it borrows from a bank using a warehouse line. The loans are pledged to the bank, and the bank lends a percentage of their value. Every few months, the lender bundles the loans into a securitization (like my Auto Loan ABS model), sells the bonds, and uses the cash to pay the warehouse line back. Then it starts filling the line up again.

Consumer Portfolio Services, a real non-prime auto lender, works exactly like this. At the end of 2025 it had three warehouse lines with $702.5M of capacity, it securitizes about four times a year, and it repays the lines from each securitization.

## II. The borrowing base

The bank does not lend against every loan. The amount it will lend is the borrowing base, and I calculated it from a 1,200-loan tape in four steps, following the structure in the UACC (Vroom) warehouse agreement from 2025:

| Step | Amount |
|---|---|
| Total pool (1,200 loans) | $28.8M |
| Less: ineligible loans (too late on payments, term too long, FICO too low, etc.) | ($3.3M) |
| Eligible pool | $25.5M |
| Less: excess concentrations | ($0.7M) |
| Adjusted eligible pool | $24.8M |
| x Blended advance rate (85% near-prime, 75% non-prime) | 82.0% |
| **Borrowing base** | **$20.3M** |
| Amount drawn today | $19.5M |
| **Available to draw** | **$0.8M** |

The excess concentration comes from Texas. Texas loans are 22.7% of the eligible pool, and the limit for any one state is 20%, so the bank will not lend against the extra $0.7M. Overall, the bank is lending about 70.6% of the total pool, even though the advance rates look like 75% to 85%. That gap was the first thing that stood out to me, because the eligibility tests and limits take away a lot more borrowing power than I expected.

## III. Liquidity over the next 12 months

I projected the pool, the borrowing base, the warehouse balance and the lender's cash month by month under four scenarios. The lender starts with $12M of cash and has to keep at least $5M to meet its liquidity covenant with the bank.

| | Base | Credit Stress | Market Freeze | Combined Stress |
|---|---|---|---|---|
| What happens | Normal | Defaults double, late loans jump to 8%, advance rates cut 10 pts | No securitizations for 9 months | Both |
| Lowest cash | $10.1M | $2.7M | ($37.0M) | ($44.3M) |
| Months below the $5M covenant | 0 | 1 | 8 | 11 |
| First month below the covenant | Never | 2 | 4 | 2 |
| Months the warehouse line was full | 0 | 0 | 8 | 7 |

## IV. What I found

The lender can handle a credit shock but not a securitization market that shuts down. In the Credit Stress case, cash dips below the covenant for one month and then recovers as soon as the next securitization goes through. In the Market Freeze case, nothing goes wrong with the borrowers at all, but the warehouse line fills up to its $40M limit in month 4. From that point on, every new loan has to be paid for entirely in cash, and the lender breaks its covenant in that same month.

This made me realize that for a lender like this, the biggest risk is not only whether borrowers pay, but whether it can keep selling its loans. In both freeze cases the model keeps making $10M of new loans a month to show how big the gap gets. In real life the lender would have to slow down. I tested how much by lowering new loans on the Inputs sheet: to stay above the covenant through a 9-month freeze, it would have to cut new lending from $10M to about $5.3M a month (almost half), and to about $3.1M a month if a credit shock hit at the same time.

## V. How the model works

- **Eligibility:** each loan in the tape is tested with formulas (more than 60 days late, original term over 75 months, LTV over 150%, FICO under 540, balance over $50,000). A loan has to pass all five.
- **Concentration limits:** any single state 20%, non-prime loans 30%, terms over 72 months 10%, LTV over 125% 15%, all as a share of the eligible pool.
- **Borrowing base:** the lesser of the blended advance rate times the adjusted eligible pool, and the $40M commitment.
- **Monthly projection:** defaults, recoveries, payments and new loans change the pool. The lender draws the full borrowing base each month, and every quarter it securitizes the eligible loans, receiving 94% of their balance minus 1% in costs and keeping the other 6% as a retained interest. Older securitizations also release cash to the lender every month, and that cash is cut in half under credit stress because those deals trap cash when losses rise.
- **Cash:** starting cash, minus new loans, plus collections, interest, recoveries, warehouse draws, securitization proceeds and cash from past deals, minus warehouse interest, fees and operating costs.

To make sure the model is right, I recalculated the borrowing base separately from the loan tape and checked every month of every scenario, and the results match.

## VI. Limitations

- The loan tape and the facility terms (size, advance rates, limits) are hypothetical. Real agreements black out their advance rates, so mine are assumptions.
- The 12-month projection works on the pool as a whole, not loan by loan, and keeps today's share of ineligible loans and excess concentrations.
- Recoveries arrive in the same month as the default, and the lender always draws the full borrowing base.
- The lender's other sources of money (corporate debt, new equity) are not modeled, which is why cash can go negative in the freeze cases.

## VII. Sources

- SOFR, October 7, 2026 (3.88%): https://www.sofrrate.com/sofr-rate/2026-10
- Credit Acceptance warehouse pricing, SOFR + 2.05% (Jul 30, 2025): https://www.ir.creditacceptance.com/node/21856 and SOFR + 1.85% (Sep 19, 2025): https://www.ir.creditacceptance.com/node/21986
- S&P Global Ratings, 2025 subprime auto ABS recoveries of 37.74%, via Auto Remarketing (Feb 16, 2026): https://www.autoremarketing.com/?p=82346
- Consumer Portfolio Services 10-K for 2025: https://www.sec.gov/Archives/edgar/data/889609/000168316826001856/cps_i10k-123125.htm
- Vroom (UACC) warehouse agreement, Exhibit 10.1: https://www.sec.gov/Archives/edgar/data/1580864/000095017025105110/vrm-ex10_1.htm

## Files

- `Warehouse_Lending_Model.xlsx`: Instructions, Summary, Inputs, Borrowing Base, Loan Tape, one sheet for each scenario, and Benchmarks
