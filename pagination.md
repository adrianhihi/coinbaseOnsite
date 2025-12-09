```java
import java.util.ArrayList;            // for dynamic array List
import java.util.Collections;          // for Collections.sort
import java.util.Comparator;           // for custom comparator
import java.util.List;                 // for List interface

public class QuerySystem {
    private final List<List<String>> records;   // original unfiltered records
    private List<List<String>> filteredRecords; // after filters + sort
    private int pageSize;                       // number of records per page
    private int currentPage;                    // zero-based page index

    // filter criteria
    private int startTime;          // inclusive lower bound for timestamp
    private int endTime;            // inclusive upper bound for timestamp
    private int minAmount;          // inclusive lower bound for amount
    private int maxAmount;          // inclusive upper bound for amount
    private String userIdFilter;    // exact match filter for user ID
    private String currencyFilter;  // exact match filter for currency

    // Constructor: initialize fields and compute the initial filteredRecords
    public QuerySystem(List<List<String>> records) {
        this.records = records;                // store master list
        this.pageSize = Integer.MAX_VALUE;     // default: no pagination
        this.currentPage = 0;                  // start at page 1 (index 0)
        this.startTime = Integer.MIN_VALUE;    // no time lower bound
        this.endTime = Integer.MAX_VALUE;      // no time upper bound
        this.minAmount = Integer.MIN_VALUE;    // no amount lower bound
        this.maxAmount = Integer.MAX_VALUE;    // no amount upper bound
        this.userIdFilter = null;              // no user filter
        this.currencyFilter = null;            // no currency filter
        this.filteredRecords = new ArrayList<>(); // init filtered list
        recomputeFiltered();                   // apply filters & sort
    }

    // Recompute filteredRecords based on current filters, then sort & reset page
    private void recomputeFiltered() {
        filteredRecords = new ArrayList<>();    // clear old results
        // apply filters
        for (List<String> rec : records) {
            int timestamp = Integer.parseInt(rec.get(0)); // parse timestamp
            String user = rec.get(2);                     // extract user ID
            String curr = rec.get(3);                     // extract currency
            int amount = Integer.parseInt(rec.get(4));    // parse amount

            // time filter
            if (timestamp < startTime || timestamp > endTime) {
                continue; // skip if outside time range
            }
            // amount filter
            if (amount < minAmount || amount > maxAmount) {
                continue; // skip if outside amount range
            }
            // user ID filter
            if (userIdFilter != null && !userIdFilter.equals(user)) {
                continue; // skip if user doesn't match
            }
            // currency filter
            if (currencyFilter != null && !currencyFilter.equals(curr)) {
                continue; // skip if currency doesn't match
            }
            filteredRecords.add(rec); // record passed all filters
        }
        // sort by timestamp ascending
        Collections.sort(filteredRecords, new Comparator<List<String>>() {
            @Override
            public int compare(List<String> a, List<String> b) {
                // compare numeric timestamp values
                return Integer.parseInt(a.get(0)) - Integer.parseInt(b.get(0));
            }
        });
        currentPage = 0; // reset to first page after any filter change
    }

    // Setter for page size; resets and recomputes pagination
    public void setPageSize(int size) {
        this.pageSize = size;    // update page size
        recomputeFiltered();     // re-filter and reset page
    }

    // Setter for time range; resets and recomputes
    public void setTimeRange(int start, int end) {
        this.startTime = start;  // set lower time bound
        this.endTime = end;      // set upper time bound
        recomputeFiltered();     // re-filter and reset page
    }

    // Setter for amount range; resets and recomputes
    public void setAmountRange(int start, int end) {
        this.minAmount = start;  // set lower amount bound
        this.maxAmount = end;    // set upper amount bound
        recomputeFiltered();     // re-filter and reset page
    }

    // Setter for user ID filter; resets and recomputes
    public void setUserId(String id) {
        this.userIdFilter = id;  // set user ID filter
        recomputeFiltered();     // re-filter and reset page
    }

    // Setter for currency filter; resets and recomputes
    public void setCurrency(String currency) {
        this.currencyFilter = currency; // set currency filter
        recomputeFiltered();            // re-filter and reset page
    }

    // Return the next page of results
    public List<List<String>> nextPage() {
        int startIndex = currentPage * pageSize; // compute slice start
        // if startIndex beyond list, no more pages
        if (startIndex >= filteredRecords.size()) {
            return new ArrayList<>();            // return empty
        }
        // compute slice end (inclusive of available records)
        int endIndex = Math.min(startIndex + pageSize, filteredRecords.size());
        // create a new list from the sublist
        List<List<String>> page = new ArrayList<>(filteredRecords.subList(startIndex, endIndex));
        currentPage++; // advance to next page for subsequent calls
        return page;   // return the page
    }

    // Return the previous page of results
    public List<List<String>> prevPage() {
        // if we're at page 1 or earlier, can't go back
        if (currentPage <= 1) {
            currentPage = 0;                   // ensure we're at first page
            return new ArrayList<>();          // return empty
        }
        // step back one page index
        currentPage -= 2;                      // move pointer to previous page
        int startIndex = currentPage * pageSize; 
        int endIndex = Math.min(startIndex + pageSize, filteredRecords.size());
        // slice out that page
        List<List<String>> page = new ArrayList<>(filteredRecords.subList(startIndex, endIndex));
        currentPage++;                         // reset currentPage to the page we just provided
        return page;                           // return the previous page
    }
}

```
