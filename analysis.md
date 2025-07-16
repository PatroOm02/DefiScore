```markdown
# DeFi Wallet Credit Scoring: Technical Analysis

This document details the technical implementation of the DeFi wallet credit scoring system, covering data preprocessing, feature engineering, and the heuristic model logic.

## 1. Data Source

The input data is a JSON file (`user-wallet-transactions.json`) containing historical DeFi transaction records. Each record includes details such as `userWallet`, `network`, `protocol`, `timestamp`, `action`, and a nested `actionData` object containing `amount`, `assetPriceUSD`, and `assetSymbol`.

## 2. Data Preprocessing and Feature Engineering (Step 2)

The raw transaction data is transformed into a structured format suitable for analysis and model building. The following sub-steps were performed:

### 2.1 Data Loading
* The JSON file is loaded into a Pandas DataFrame. Initial inspection confirms the presence of `100,000` transaction records.

### 2.2 Data Cleaning and Type Conversion
* The `timestamp` column, initially in Unix seconds, is converted to datetime objects for easier time-series analysis.
* Crucially, nested fields within the `actionData` column (`amount`, `assetPriceUSD`, `assetSymbol`) are extracted into their own top-level columns (`actionData.amount`, `actionData.assetPriceUSD`, `actionData.assetSymbol`). This is done using a `lambda` function with `.get()` to safely handle cases where these keys might be missing or `actionData` is not a dictionary.
* `actionData.amount` and `actionData.assetPriceUSD` are converted to numeric (`float64`), with `errors='coerce'` to turn any non-convertible values into `NaN`.
* Rows with `NaN` values in `actionData.amount` or `actionData.assetPriceUSD` (indicating essential missing data for calculation) are dropped. In this dataset, `0` rows were dropped, retaining all `100,000` transactions.

### 2.3 Derive USD Values
* A new column, `amount_usd`, is created by multiplying `actionData.amount` by `actionData.assetPriceUSD`. This normalizes transaction values to a common currency, making them comparable regardless of the original asset.

### 2.4 Feature Grouping (Per Wallet)
* The cleaned transaction data is grouped by `userWallet` to create aggregate features for each unique wallet. This results in a `wallet_features` DataFrame where each row represents a distinct wallet (`3497` unique wallets found).
* **Engineered Features include:**
    * **Activity Metrics:**
        * `total_transactions`: Total number of transactions for the wallet.
        * `num_unique_actions`: Number of distinct action types (e.g., 'deposit', 'borrow', 'repay').
        * `[action_type]_count`: Specific counts for each action type (e.g., `deposit_count`, `borrow_count`, `repay_count`, `redeemunderlying_count`).
    * **Temporal Features:**
        * `first_transaction_timestamp`: Timestamp of the earliest transaction.
        * `last_transaction_timestamp`: Timestamp of the latest transaction.
        * `wallet_age_days`: Duration between the first and last transaction (indicating activity longevity).
    * **Volume Metrics (in USD):**
        * `total_deposited_usd`: Sum of all deposited amounts in USD.
        * `total_borrowed_usd`: Sum of all borrowed amounts in USD.
        * `total_repaid_usd`: Sum of all repaid amounts in USD.
        * `total_redeemed_usd`: Sum of all redeemed amounts in USD.
        * `net_deposit_borrow_usd`: `total_deposited_usd - total_borrowed_usd` (indicates net inflow/outflow).
        * `avg_deposit_usd`, `avg_borrow_usd`, `avg_repay_usd`: Average amounts for these transaction types.
    * **Behavioral Ratios & Indicators:**
        * `repayment_ratio`: `total_repaid_usd / total_borrowed_usd`. A value of 1.0 indicates full repayment (or no borrows). Handled division by zero by setting to 1.0 if no borrows.
        * `num_liquidations`: Renamed from `liquidationcall_count` (indicates instances where collateral was seized due to insufficient funds).
        * `redeem_to_deposit_ratio`: `total_redeemed_usd / total_deposited_usd`. Indicates how much a user withdraws relative to their deposits. Handled division by zero by setting to 0 if no deposits.
        * `num_unique_assets`: Number of distinct assets involved in the wallet's transactions.
    * All `NaN` values resulting from initial grouping (e.g., a wallet never borrowed, so `total_borrowed_usd` would be `NaN`) are filled with `0`.

## 3. Model Selection and Implementation (Step 3)

Given the absence of pre-labeled credit scores, a **heuristic-based scoring model** was implemented. This approach provides transparency and direct control over the scoring logic.

### 3.1 Heuristic Rule Definition and Calculation
* **Feature Scaling:** All relevant numerical features (excluding `num_liquidations` for direct penalty application) are scaled to a `[0, 1]` range using `MinMaxScaler`. This ensures that features with larger numerical ranges do not disproportionately influence the score.
* **Base Score:** Each wallet starts with a `base_score` of `500`.
* **Positive Contributors (Score Increase):**
    * `repayment_ratio`: Heavily weighted (0.4 * score_range) as a primary indicator of reliability.
    * `total_deposited_usd`: Significant weight (0.2 * score_range) for higher overall deposits.
    * `wallet_age_days`: Moderate weight (0.1 * score_range) for sustained activity.
    * `deposit_count`, `num_unique_actions`, `num_unique_assets`: Smaller positive contributions (0.05 * score_range each) for active and diversified engagement.
* **Negative Contributors (Score Decrease):**
    * `num_liquidations`: A strong direct penalty (`-200` points per liquidation event). Each liquidation severely impacts the score.
    * `redeem_to_deposit_ratio`: Moderate negative weight (0.2 * score_range) for high redemption relative to deposits, potentially indicating less stable behavior.
* **Clamping:** The `raw_credit_score` is then clamped between `0` and `1000` to ensure it fits the required score range.

### Score Distribution Overview:
* Minimum Score: 0.00
* Maximum Score: 725.10
* Mean Score: 556.98

## 4. One-Step Script Implementation (Step 4)

All the above steps are encapsulated within a single Python script (or a single Jupyter Notebook cell) that can be executed to generate the wallet scores. The script takes the input JSON file path and an output CSV file path as arguments (or defined variables within the notebook).

## 5. Conclusion

This project successfully demonstrates a methodology for assigning credit scores to DeFi wallets based on their on-chain transaction history. The heuristic model provides a transparent and interpretable scoring mechanism, allowing for assessment of wallet behavior from reliable participants to potentially risky entities. The generated `wallet_scores.csv` can be used for further analysis, risk management, or integration into DeFi applications.

---
