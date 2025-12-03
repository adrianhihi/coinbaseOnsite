
# Currency Converter – DFS 解法 & 优化版（带剪枝）

> 说明：  
> - 题意：给定一组货币汇率（双向），求从 `from` 到 `to` 的**最大可达汇率**（路径上边权相乘最大）。  
> - 解法：把货币抽象成图，用 DFS 搜索所有简单路径，取乘积最大的一条。  
> - 下面给出两个版本：
>   1. **Part 1：基础版 DFS**（和参考答案等价）  
>   2. **Part 2：在 Part 1 基础上做最小改动，加剪枝优化**（标出所有修改处）

---

## Part 1：基础版 DFS 解法（参考答案版）

### 思路

- 使用邻接表建图：  
  - 每条关系 `fromArr[i] -> toArr[i]` 汇率为 `rateArr[i]`；  
  - 同时加一条反向边 `toArr[i] -> fromArr[i]`，汇率为 `1 / rateArr[i]`。
- 查询时，从 `from` 开始做 DFS：  
  - 参数 `rate` 记录从起点到当前节点的累计汇率（路径上所有边乘积）；  
  - 到达 `target` 时返回当前乘积；  
  - 多条路径中取最大值。
- `visited` 集合用来保证每条路径上不会重复访问同一个节点（避免环）。

### 代码（基础版 DFS）

```java
import java.util.*;

class Pair {
    String to;
    double rate;

    public Pair(String to, double rate) {
        this.to = to;
        this.rate = rate;
    }
}

class CurrencyConverter {
    private Map<String, List<Pair>> map;

    public CurrencyConverter(String[] fromArr, String[] toArr, double[] rateArr) {
        map = new HashMap<>();
        for (int i = 0; i < fromArr.length; i++) {
            String fromCurrency = fromArr[i];
            String toCurrency = toArr[i];
            double rate = rateArr[i];

            map.putIfAbsent(fromCurrency, new ArrayList<>());
            map.putIfAbsent(toCurrency, new ArrayList<>());

            // fromCurrency -> toCurrency with rate
            map.get(fromCurrency).add(new Pair(toCurrency, rate));
            // toCurrency -> fromCurrency with reciprocal rate
            map.get(toCurrency).add(new Pair(fromCurrency, 1 / rate));
        }
    }

    public double getBestRate(String from, String to) {
        // If either currency is not present, no path exists
        if (!map.containsKey(from) || !map.containsKey(to)) {
            return -1; // basic version uses -1
        }

        Set<String> visited = new HashSet<>();
        // Start DFS with product = 1
        return dfs(from, to, 1, visited);
    }

    private double dfs(String current, String target, double rate, Set<String> visited) {
        // If we reached the target, current product is a candidate answer
        if (current.equals(target)) {
            return rate;
        }

        visited.add(current);
        double maxRate = -1;

        if (map.containsKey(current)) {
            for (Pair pair : map.get(current)) {
                if (visited.contains(pair.to)) {
                    // Avoid cycles on the current path
                    continue;
                }

                // Explore next currency with updated product
                double result = dfs(pair.to, target, rate * pair.rate, visited);
                maxRate = Math.max(maxRate, result);
            }
        }

        // Backtrack: remove from visited so other paths can use this node
        visited.remove(current);
        return maxRate;
    }
}
```

### 复杂度（基础版）

- 设：  
  - `V` = 货币种类数（节点数）  
  - `E` = 边数（约为 `2 * fromArr.length`）
- 构造函数建图：时间 O(E)，空间 O(V + E)。
- `getBestRate` + DFS：
  - 理论上会枚举所有**简单路径**（路径上不重复节点），简单路径数量在一般图中可能是指数级；
  - 因此**最坏时间复杂度**可能是指数级（例如 O(2^V) 量级）。  
  - 空间复杂度：O(V + E)（图） + O(V)（递归栈和 visited）。

---

## Part 2：优化版 DFS（在基础版上加剪枝）

在基础版 DFS 上，我们增加一个**剪枝优化**：

- 新增一个 `bestFromStart` 哈希表，记录：  
  `bestFromStart.get(x)` =「从起点 `from` 到货币 `x` 的**当前已知最大乘积**」。  
- 在 DFS 中每次到达某个 `current` 时：  
  - 如果这次的 `rate` **不大于**我们之前已经记录过的最佳乘积，说明：  
    - 之前我们用一条“更强的路径前缀”到达过 `current`；  
    - 再从这个更弱的前缀继续走，不可能打败之前的结果；  
    - 因此可以直接剪枝（返回 -1），不再继续 DFS。

### 相对于 Part 1 的修改点（用注释标出）

1. `getBestRate` 中：
   - 新增 `Map<String, Double> bestFromStart = new HashMap<>();`  
   - 调用 `dfs` 时多传一个参数 `bestFromStart`。  
   - 顺便把返回值统一成 `-1.0`（double 字面量）。

2. `dfs` 方法签名修改：
   - 从 `dfs(String current, String target, double rate, Set<String> visited)`  
   - 改为 `dfs(String current, String target, double rate, Set<String> visited, Map<String, Double> bestFromStart)`。

3. `dfs` 内部增加剪枝逻辑：
   - 在递归展开邻居之前，增加：
     ```java
     double bestSeen = bestFromStart.getOrDefault(current, 0.0);
     if (rate <= bestSeen) {
         return -1.0;
     }
     bestFromStart.put(current, rate);
     ```

### 代码（优化版 DFS，所有修改处已标注）

```java
import java.util.*;

class Pair {
    String to;
    double rate;

    public Pair(String to, double rate) {
        this.to = to;
        this.rate = rate;
    }
}

class CurrencyConverter {
    private Map<String, List<Pair>> map;

    public CurrencyConverter(String[] fromArr, String[] toArr, double[] rateArr) {
        map = new HashMap<>();
        for (int i = 0; i < fromArr.length; i++) {
            String fromCurrency = fromArr[i];
            String toCurrency = toArr[i];
            double rate = rateArr[i];

            map.putIfAbsent(fromCurrency, new ArrayList<>());
            map.putIfAbsent(toCurrency, new ArrayList<>());

            // fromCurrency -> toCurrency with rate
            map.get(fromCurrency).add(new Pair(toCurrency, rate));
            // toCurrency -> fromCurrency with reciprocal rate
            map.get(toCurrency).add(new Pair(fromCurrency, 1 / rate));
        }
    }

    public double getBestRate(String from, String to) {
        // [NEW] Basic null check (defensive, optional)
        if (from == null || to == null) {
            return -1.0; // [NEW]
        }

        // If either currency is not present, no path exists
        if (!map.containsKey(from) || !map.containsKey(to)) {
            return -1.0; // [CHANGED] use -1.0 as double literal
        }

        // [NEW] Optional short-circuit: same currency => rate 1.0
        if (from.equals(to)) {
            return 1.0; // [NEW]
        }

        Set<String> visited = new HashSet<>();
        // [NEW] bestFromStart used for pruning
        Map<String, Double> bestFromStart = new HashMap<>(); // [NEW]

        // [CHANGED] dfs signature now takes bestFromStart
        return dfs(from, to, 1.0, visited, bestFromStart); // [CHANGED]
    }

    // [CHANGED] added Map<String, Double> bestFromStart parameter
    private double dfs(String current,
                       String target,
                       double rate,
                       Set<String> visited,
                       Map<String, Double> bestFromStart) { // [CHANGED SIGNATURE]

        // If we reached the target, current product is a candidate answer
        if (current.equals(target)) {
            return rate;
        }

        // [NEW] Pruning: if we have visited `current` before with an equal or better product,
        // continuing from here cannot yield a better overall result.
        double bestSeen = bestFromStart.getOrDefault(current, 0.0); // [NEW]
        if (rate <= bestSeen) { // [NEW]
            return -1.0;        // [NEW]
        }
        bestFromStart.put(current, rate); // [NEW]

        visited.add(current);
        double maxRate = -1.0; // [CHANGED] use -1.0 as double literal

        if (map.containsKey(current)) {
            for (Pair pair : map.get(current)) {
                if (visited.contains(pair.to)) {
                    // Avoid cycles on the current path
                    continue;
                }

                // Explore next currency with updated product
                double result = dfs(
                    pair.to,
                    target,
                    rate * pair.rate,
                    visited,
                    bestFromStart   // [CHANGED] pass bestFromStart down
                );
                maxRate = Math.max(maxRate, result);
            }
        }

        // Backtrack: remove from visited so other paths can use this node
        visited.remove(current);
        return maxRate;
    }
}
```

### 复杂度（优化版）

- 构造函数：同基础版，O(E) 时间、O(V + E) 空间。
- `getBestRate` + DFS（带剪枝）：
  - 理论 worst-case：在某些“极端密图”里，简单路径数量依然可能指数级，因此最坏复杂度仍可能指数级；
  - 但 `bestFromStart` 剪枝会在实践中砍掉大量“明显不可能更优”的分支：
    - 对于同一个节点 `current`，只会保留“从起点到这里乘积最大”的若干路径前缀；
    - 比起基础版 DFS，一般情况下会大幅减少搜索空间。
  - 空间复杂度：  
    - 图：O(V + E)  
    - 递归栈 + visited + bestFromStart：O(V)

---

## 面试时的讲法小抄（简短版）

1. **第一版（基础 DFS）**  
   - 把每个货币看成图的节点，每个汇率是一条带权边；因为汇率是双向的，所以每条输入对应两条边。  
   - 查询时，用 DFS 从 `from` 搜到 `to`，路径上的边权相乘，取所有简单路径中最大乘积。  
   - 用 `visited` 防止在同一条路径里进入环。

2. **优化版（在 DFS 上加剪枝）**  
   - 我进一步加了一个 `bestFromStart` map，记住「从起点到某个节点的当前最大乘积」。  
   - 如果某次 DFS 到达同一个节点时乘积更小，就马上剪枝，因为这条前缀不可能击败之前那条更强的前缀。  
   - 在保持结果正确的前提下显著减少搜索空间，尤其在图比较大的时候更高效。

可以先在白板上写出 Part 1 这种基础版 DFS，然后和面试官说：
> “如果数据量稍微大一点，我会在 DFS 的基础上加一个 pruning map 去剪掉明显不可能更优的路径”，  
然后把 Part 2 改动几行写出来（修改点都已经在上面代码里标好了）。
