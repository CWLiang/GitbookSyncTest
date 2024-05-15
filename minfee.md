# Minimum Required Fee Machanism

## Primary Borrow Order

The tx fee will be charged from the received borrow amount.

### Objective

The borrow amount specified by a user needs to be 

### Frontend

1.  Calc `signedMinFeeAmt0`
1.  Calc `minBorrowAmt = signedMinFeeAmt0`
1.  Constrain `inputBorrowAmt >= minBorrowAmt`

### Backend

1.  Calc `expectedMinFeeAmt0`
1.  Check if `borrowAmt >= signedMinFeeAmt0`
1.  Check if `coda >= expectedMinFeeAmt0 - signedMinFeeAmt0`

## Auction Lend

We will charge fee from additionally locked amt.

### Frontend

To calc the upper bound of `maxLendAmtByMinFee`
1.  Calc `signedMinFeeAmt1`
1.  Calc `maxLendAmtByMinFee = avlAmt - signedMinFeeAmt1`

### Backend

1.  Calc `expectedMinFeeAmt1`
1.  Check if `avlAmt >= signedLendAmt + signedMinFeeAmt1`
1.  Check if `coda >= expectedMinFeeAmt1 - signedMinFeeAmt1`

## Secondary Limit Sell

We will charge fee from received amount.

### Frontend

1.  Calc `signedMinFeeAmt0`, `signedMinFeeAmt1`
1.  Calc `maxMinFeeAmt = max(signedMinFeeAmt0, signedMinFeeAmt1)`
1.  Calc `days := (interestRate >= 0) ? dOTM : dETM`
1.  Calc `minMQByMinFee` with the formula
    $$
        \lceil \frac{minFeeAmt_{max} \times (365 + interestRate \times days)}{365} \rceil
    $$

### Backend

1.  Calc `expectedMinFeeAmt0`, `expectedMinFeeAmt1`
1.  Calc `maxMatchedBQ`
1.  Check if `coda >= maxMatchedBQ - Max(signedMinFeeAmt0, signedMinFeeAmt1)`

## Secondary Limit Buy

We will charge fee from additionally locked amt.

### Frontend

1.  Calc `signedMinFeeAmt0`, `signedMinFeeAmt1`
1.  Calc `maxMinFeeAmt = max(signedMinFeeAmt0, signedMinFeeAmt1)`
1.  Calc `actualAvlAmt = avlAmt - maxMinFeeAmt`
1.  Calc `days := (interestRate >= 0) ? dETM : dOTM`
1.  Calc `maxBQbyMinFee` with the formula
    $$
        \lfloor \frac{actualAvlAmt \times (365 + interestRate \times days)}{365} \rfloor
    $$

### Backend

1.  Calc `expectedMinFeeAmt0`, `expectedMinFeeAmt1`
1.  Calc `maxMatchedBQ`
1.  Check if `avlAmt >= maxMatchedBQ + max(signedMinFeeAmt0, signedMinFeeAmt1)`
1.  Check if `coda >= max(expectedMinFeeAmt0, expectedMinFeeAmt1) - max(signedMinFeeAmt0, signedMinFeeAmt1`
