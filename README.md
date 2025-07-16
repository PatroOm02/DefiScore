# DeFi Wallet Credit Scoring Project

This project provides a system to assign a credit score (0-1000) to DeFi (Decentralized Finance) wallets based on their historical transaction behavior. The goal is to identify reliable and responsible wallet usage versus risky or potentially exploitative behavior.

## Project Purpose

In the decentralized finance ecosystem, traditional credit scores do not exist. This project aims to bridge that gap by analyzing on-chain transaction data to assess the "creditworthiness" of a wallet. This can be used for various applications, such as:
* **Risk assessment:** Identifying high-risk borrowers or participants.
* **User segmentation:** Grouping users by their behavioral patterns.
* **Incentive mechanisms:** Rewarding users with high scores.
* **Undercollateralized lending (future potential):** Enabling new forms of lending based on reputation.

## How it Works

The system processes raw JSON transaction data, extracts relevant features, and applies a heuristic model to calculate a credit score for each unique wallet.

## Setup and Execution

### Prerequisites

* Python 3.x
* pandas
* numpy
* scikit-learn

You can install the required libraries using pip:
```bash
pip install pandas numpy scikit-learn
