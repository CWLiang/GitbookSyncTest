# Minimum Required Fee Machanism

## Primary Borrow Order

The tx fee will be charged from the received borrow amount.

### Frontend

1.  Calculate `signedMinFeeAmt0`
    *   $$signedMinFeeAmt0 = \lceil \frac{primaryBorrowMinFeeAmt*price_{ETH}}{price_{borrowToken}} \lceil$$

1.  Constrain `userInputBorrowAmt >= signedMinFeeAmt0`


### Backend (API)

1.  Calculate `expectedMinFeeAmt0`
    *   $$expectedMinFeeAmt0= \lceil \frac{primaryBorrowMinFeeAmt*price_{ETH}}{price_{borrowToken}} \rceil $$
1.  Check `signedBorrowAmt >= signedMinFeeAmt0`
1.  Check `expectedMinFeeAmt0 - signedMinFeeAmt0 < acceptableVariance`

## Primary Lend Order

The tx fee will be charged from the locked amount.

### Frontend

#### Lower Bound

1.  Calculate `minFeeAmt0`
    *   > The lower bound of a lend order ensures that the borrowed amount, when matched with a borrow order, is sufficient for the borrower to cover the minimum required fee. Therefore, we use the current `minFeeAmt0` as constraint.
    *   $$signedMinFeeAmt0 = \lceil \frac{primaryBorrowMinFeeAmt*price_{ETH}}{price_{borrowToken}} \lceil$$
1.  Constrain `userInputLendAmt >= minFeeAmt0`

#### Upper Bound

1.  Calculate `signedMinFeeAmt1`
    *   $$signedMinFeeAmt1 = \lceil \frac{primaryLendMinFeeAmt*price_{ETH}}{price_{lendToken}} \rceil $$
1.  Calculate `maxLendAmtByMinFee`
    *   $$maxLendAmtByMinFee = avlAmt - signedMinFeeAmt1$$
1.  Calculate `maxLendAmtByTxFee`
    *   $$maxLendAmtByTxFee=\lfloor \frac{365*available}{365+defaultMatchedInterestRate*lendFeeRate*d_{OTM}} \rfloor$$
    *   > $lendTxFeeAmt = lendAmt * defaultMatchedInterestRate * lendFeeRate * \frac{d_{OTM}}{{365}} \\ \Rightarrow available - maxLendAmtByTxFee = maxLendAmtByTxFee*defaultMatchedInterestRate*lendFeeRate*\frac{d_{OTM}}{365} \\ \Rightarrow available = maxLendAmtByTxFee * (1+defaultMatchedInterestRate*lendFeeRate*\frac{d_{OTM}}{365}) \\ \Rightarrow available = maxLendAmtByTxFee * \frac{365+defaultMatchedInterestRate*lendFeeRate*d_{OTM}}{365} \\ \Rightarrow maxLendAmtByTxFee = \frac{365*available}{365+defaultMatchedInterestRate*lendFeeRate*d_{OTM}}$
1.  Calculate `maxLendAmt`
    *   $$maxLendAmt := min(maxLendAmtByTxFee, maxLendAmtByMinFee)$$
1.  Constrain `userInputLendAmt <= maxLendAmt`

### Backend (API)

1.  Calculate `expectedMinFeeAmt1`
    *   $$expectedMinFeeAmt1=\frac{primaryBorrowMinFeeAmt*price_{ETH}}{price_{lendToken}}$$
1.  Calculate `derivedTxFeeAmt`
    *   $$derivedTxFeeAmt = \frac{signedLendAmt * defaultMatchedInterestRate * lendFeeRate * d_{OTM}}{365}$$
1.  Calculate `maxFeeAmt`
    *   $$maxFeeAmt := max(derivedMinFeeAmt1, signedTxFeeAmt)$$
1.  Check `avlAmt >= signedLendAmt + maxMinFeeAmt`
1.  Check `expectedMinFeeAmt1 - signedMinFeeAmt1 < acceptableVariance`

## Secondary Limit Sell Order

The tx fee will be charged from the received buy amount (base token).

### Frontend

1.  Calculate `signedMinFeeAmt0`, `signedMinFeeAmt1`
    *   $$
            signedMinFeeAmt0 = \lceil \frac{secondaryTakerMinFeeAmt*price_{ETH}}{price_{baseToken}} \rceil
        $$
    *   $$
            signedMinFeeAmt1 = \lceil \frac{secondaryMakerMinFeeAmt*price_{ETH}}{price_{baseToken}} \rceil
        $$
1.  Calculate `maxMinFeeAmt`
    *   $$
            maxMinFeeAmt := max(signedMinFeeAmt0, signedMinFeeAmt1)
        $$
1.  Calculate `dayToMaturity`
    *   $$
            dayToMaturity := (interestRate \geq 0) ? d_{OTM} : d_{ETM}
        $$
1.  Calculate `minSellAmtByMinFee` 
    *   $$
            minSellAmtByMinFee = \lceil \frac{maxMinFeeAmt \times (365 + interestRate \times dayToMaturity)}{365} \rceil
        $$
1.  Constrain `userInputSellAmt >= minSellAmtByMinFee`

### Backend

1.  Calculate `expectedMinFeeAmt0`, `expectedMinFeeAmt1`
    *   $$
            expectedMinFeeAmt0 = \lceil \frac{secondaryTakerMinFeeAmt*price_{ETH}}{price_{baseToken}} \rceil
        $$
    *   $$
            expectedMinFeeAmt1 = \lceil \frac{secondaryMakerMinFeeAmt*price_{ETH}}{price_{baseToken}} \rceil
        $$   
1.  Calculate `maxExpectedMinFeeAmt`
    *   $$
            maxExpectedMinFeeAmt = max(expectedMinFeeAmt0, expectedMinFeeAmt1)
        $$
1.  Calculate `dayToMaturity`
    *   $$
            dayToMaturity := (signedSellAmt \geq signedBuyAmt) ? d_{OTM} : d_{ETM}
        $$
1.  Calculate `minMatchedBuyAmt`
    *   $$
            minMatchedBuyAmt = \lfloor signedSellAmt * (1 + (\frac{signedSellAmt}{signedBuyAmt}-1)*\frac{d_{dayToMaturity}}{365}) \rfloor
        $$
1.  Check `minMatchedBuyAmt - maxExpectedMinFeeAmt < acceptableVariance`
1.  Check `expectedMinFeeAmt0 - signedMinFeeAmt0 < acceptableVariance`
1.  Check `expectedMinFeeAmt1 - signedMinFeeAmt1 < acceptableVariance`

## Secondary Limit Buy Order

The tx fee will be charged from the locked amount.

### Frontend

1.  Calculate `signedMinFeeAmt0`, `signedMinFeeAmt1`
    *   $$
            signedMinFeeAmt0 = \lceil \frac{secondaryTakerMinFeeAmt*price_{ETH}}{price_{baseToken}} \rceil
        $$
    *   $$
            signedMinFeeAmt1 = \lceil \frac{secondaryMakerMinFeeAmt*price_{ETH}}{price_{baseToken}} \rceil
        $$
1.  Calculate `maxMinFeeAmt`
    *   $$
            maxMinFeeAmt := max(signedMinFeeAmt0, signedMinFeeAmt1)
        $$
1.  Calculate `maxSellAmtByMinFee` (base token)
    *   $$
            maxBuyAmtByMinFee = avlAmt - maxMinFeeAmt
        $$
1.  Calculate `dayToMaturity`
    *   $$
            dayToMaturity =  := (interestRate \geq 0) ? d_{ETM} : d_{OTM}
        $$
1.  Calculate `maxBuyAmtByMinFee` 
    *   $$
            maxBuyAmtByMinFee = \lfloor \frac{maxSellAmtByMinFee \times (365 + interestRate \times dayToMaturity)}{365} \rfloor
        $$

### Backend

1.  Calculate `expectedMinFeeAmt0`, `expectedMinFeeAmt1`
    *   $$
            expectedMinFeeAmt0 = \lceil \frac{secondaryTakerMinFeeAmt*price_{ETH}}{price_{baseToken}} \rceil
        $$
    *   $$
            expectedMinFeeAmt1 = \lceil \frac{secondaryMakerMinFeeAmt*price_{ETH}}{price_{baseToken}} \rceil
        $$    
1.  Calculate `maxMatchedBQ`
1.  Check if `avlAmt >= maxMatchedBQ + max(signedMinFeeAmt0, signedMinFeeAmt1)`
1.  Check if `coda >= max(expectedMinFeeAmt0, expectedMinFeeAmt1) - max(signedMinFeeAmt0, signedMinFeeAmt1`
