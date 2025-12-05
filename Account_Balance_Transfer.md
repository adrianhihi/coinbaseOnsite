```java
import java.util.*;
import java.math.*;

class TransactionSystem {
    private Map<String, Double> balances;

    public TransactionSystem(List<List<String>> transactions, List<List<String>> accounts) {
        balances = new HashMap<>();
        for (List<String> account : accounts) {
            String name = account.get(0);
            double balance = Double.parseDouble(account.get(1));
            balances.put(name, balance);
        }

        // Process each transaction
        for (List<String> txn : transactions) {
            String from = txn.get(0);
            String to = txn.get(1);
            String percentStr = txn.get(2);

            double percentage = Double.parseDouble(percentStr.replace("%", "")) / 100;
            double transferAmount = balances.get(from) * percentage;

            balances.put(from, balances.get(from) - transferAmount);
            balances.put(to, balances.get(to) + transferAmount);
        }
    }

    public List<Double> getBalances() {
        List<String> sortedKeys = new ArrayList<>(balances.keySet());
        Collections.sort(sortedKeys);

        List<Double> res = new ArrayList<>();
        for (String key : sortedKeys) {
            res.add(balances.get(key));
        }
        return res;
    }
}
```
Follow-up
```java
import java.util.*;
import java.math.*;

/**
 * TransactionSystem
 *
 * Part 1:
 *  - Apply a list of percentage-based transfers between accounts.
 *
 * Follow-up:
 *  - Support rolling back a specific transaction by index.
 *  - After rollback, the final balances should be equivalent to
 *    re-running all remaining (non-rolled-back) transactions in order.
 *
 * Algorithm idea (follow-up):
 *  - We keep a snapshot list `states`, where:
 *      states.get(i) = balances AFTER applying transaction i-1
 *      states.get(0) = initial balances
 *  - We also keep a boolean array `rolledBack[i]` to mark deleted transactions.
 *  - When rolling back transaction idx:
 *      1) Mark rolledBack[idx] = true
 *      2) Recompute snapshots from idx onward using processTransactionsFrom(idx)
 *  - This allows us to reuse the prefix [0..idx-1] and only recompute the suffix.
 */
class TransactionSystem {
    // NEW: store original transactions
    private final List<List<String>> transactions;
    // NEW: initial balances (never mutated)
    private final Map<String, Double> initialBalances;
    // NEW: state snapshots; states.get(i) = balances after i-1 th transaction, states.get(0) = initial
    private List<Map<String, Double>> states;
    // NEW: mark whether each transaction has been rolled back
    private final boolean[] rolledBack;

    /**
     * Constructor
     *
     * Responsibilities:
     *  - Copy and store the transaction list.
     *  - Build initialBalances from the accounts input.
     *  - Initialize the rollback flags (all false).
     *  - Initialize `states` with the initial balance snapshot at index 0.
     *  - Precompute all snapshots assuming no rollback yet.
     *
     * Algorithm:
     *  - O(A + T * K) where:
     *      A = number of accounts,
     *      T = number of transactions,
     *      K = average number of accounts per snapshot copy.
     *  - For each transaction, we create a new balance map based on the previous one,
     *    then apply the percentage transfer if not rolled back.
     */
    public TransactionSystem(
        List<List<String>> transactions, 
        List<List<String>> accounts
    ) {
        // NEW: keep a copy of transactions for later rollback recomputation
        this.transactions = new ArrayList<>(transactions);

        // NEW: build initialBalances instead of a single mutable balances map
        this.initialBalances = new HashMap<>();
        for (List<String> account : accounts) {
            String name = account.get(0);
            double balance = Double.parseDouble(account.get(1));
            initialBalances.put(name, balance);
        }

        int n = transactions.size();
        this.rolledBack = new boolean[n]; // NEW: all false initially

        // NEW: initialize states; states.get(0) = snapshot of initial balances
        this.states = new ArrayList<>();
        this.states.add(new HashMap<>(initialBalances));

        // NEW: precompute all states with all transactions applied
        processTransactionsFrom(0);
    }

    /**
     * processTransactionsFrom
     *
     * Responsibilities:
     *  - Recompute balance snapshots starting from a given transaction index.
     *  - Assumes states.get(startIdx) is the snapshot BEFORE applying transaction startIdx.
     *  - After running, states.get(i+1) will be the snapshot after transaction i,
     *    for all i >= startIdx.
     *
     * Algorithm:
     *  - If startIdx == 0, we rebuild `states` entirely from initialBalances.
     *  - Otherwise, we:
     *      1) Truncate states to keep snapshots up to index startIdx.
     *      2) For each transaction i >= startIdx:
     *          - Copy the previous snapshot (HashMap copy).
     *          - If transaction i is NOT rolled back, apply percentage transfer.
     *          - Append the new snapshot to `states`.
     *  - Time complexity: O((T - startIdx) * K),
     *      where T is total number of transactions,
     *      and K is the number of accounts per snapshot copy.
     */
    // NEW: helper to recompute states from startIdx onward
    private void processTransactionsFrom(int startIdx) {
        // NEW: if recomputing from scratch, rebuild states from initialBalances
        if (startIdx == 0) {
            states = new ArrayList<>();
            states.add(new HashMap<>(initialBalances));
        }

        // NEW: truncate states so that states.get(startIdx) is the last snapshot kept
        while (states.size() > startIdx + 1) {
            states.remove(states.size() - 1);
        }

        // NEW: rebuild snapshots from startIdx to end
        Map<String, Double> prevState = states.get(startIdx);
        for (int i = startIdx; i < transactions.size(); i++) {
            Map<String, Double> currentState = new HashMap<>(prevState);

            if (!rolledBack[i]) {
                List<String> txn = transactions.get(i);
                String from = txn.get(0);
                String to = txn.get(1);
                String percentStr = txn.get(2);

                double percentage = Double.parseDouble(percentStr.replace("%", "")) / 100.0;
                double fromBalance = currentState.get(from);
                double transferAmount = fromBalance * percentage;

                currentState.put(from, fromBalance - transferAmount);
                currentState.put(to, currentState.get(to) + transferAmount);
            }
            // If rolled back, just keep currentState same as prevState

            states.add(currentState);
            prevState = currentState;
        }
    }

    /**
     * getBalances
     *
     * Responsibilities:
     *  - Return the final balances for all accounts, sorted by account name.
     *
     * Algorithm:
     *  - Take the last snapshot in `states` (i.e. after all non-rolled-back transactions).
     *  - Sort account names lexicographically.
     *  - Collect balances in that sorted order.
     *  - Time complexity:
     *      - O(A log A) for sorting, where A is number of accounts.
     */
    public List<Double> getBalances() {
        // CHANGED: use last snapshot in states instead of a single balances map
        Map<String, Double> lastState = states.get(states.size() - 1);
        List<String> sortedKeys = new ArrayList<>(lastState.keySet());
        Collections.sort(sortedKeys);

        List<Double> res = new ArrayList<>();
        for (String key : sortedKeys) {
            res.add(lastState.get(key));
        }
        return res;
    }

    /**
     * rollbackTransfer
     *
     * Responsibilities:
     *  - Logically delete the i-th transaction (0-based index).
     *  - Ensure calling rollback on the same index multiple times has no extra effect.
     *  - After rollback, recompute all snapshots from this index onward.
     *
     * Algorithm:
     *  - If transaction idx is already rolled back, return immediately.
     *  - Otherwise:
     *      1) Mark rolledBack[idx] = true.
     *      2) Call processTransactionsFrom(idx), which reuses the prefix [0..idx-1]
     *         and rebuilds the suffix [idx..T-1].
     *  - Time complexity per rollback:
     *      - O((T - idx) * K), where T is number of transactions
     *        and K is number of accounts per snapshot.
     */
    public void rollbackTransfer(int idx) {
        // NEW: ignore if already rolled back
        if (rolledBack[idx]) {
            return;
        }
        rolledBack[idx] = true;

        // NEW: recompute states from this index onward
        processTransactionsFrom(idx);
    }
}
```
