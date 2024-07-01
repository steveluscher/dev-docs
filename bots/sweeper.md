# Sweeper

## Overview

1. Monitors new incoming transactions on several different chains with getNewPendingTransaction subscriptions, filtered by your compromised wallet address as the receiver.
2. Checks token balance on the incoming transaction's chain to determine if gas fees are needed to sweep the wallet and send to your new wallet address.
3. Wallet is funded by your designated Funding Wallet (if necessary) and several transactions are quickly sent to attempt the sweep until balance is changed.
