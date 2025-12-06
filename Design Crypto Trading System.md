Version 1
```java
import java.util.*;
import java.math.*;

enum State {
    LIVE, PAUSED, COMPLETED, CANCELED
}

class Order {
    String id;
    String currency;
    int amount;
    long timestamp;
    String type;
    State state;

    public Order(String id, String currency, int amount, long timestamp, String type) {
        this.id = id;
        this.currency = currency;
        this.amount = amount;
        this.timestamp = timestamp;
        this.type = type;
        this.state = State.LIVE;
    }
}

class CryptoTradingSystem {
    private Map<State, Map<String, Order>> orderMap;

    public CryptoTradingSystem() {
        orderMap = new EnumMap<>(State.class);
        for (State state : State.values()) {
            orderMap.put(state, new HashMap<>());
        }
    }

    public String placeOrder(String id, String currency, int amount, int timestamp, String type) {
        if (findOrder(id) != null) {
            return "";
        }

        Order order = new Order(id, currency, amount, timestamp, type);
        orderMap.get(State.LIVE).put(id, order);
        return id;
    }

    public String pauseOrder(String id) {
        Order order = orderMap.get(State.LIVE).remove(id);
        if (order == null) {
            return "";
        }
        order.state = State.PAUSED;
        orderMap.get(State.PAUSED).put(id, order);
        return id;
    }

    public String resumeOrder(String id) {
        Order order = orderMap.get(State.PAUSED).remove(id);
        if (order == null) {
            return "";
        }
        order.state = State.LIVE;
        orderMap.get(State.LIVE).put(id, order);
        return id;
    }

    public String cancelOrder(String id) {
        State currentState = getOrderState(id);
        if (currentState == null || currentState == State.COMPLETED || currentState == State.CANCELED) {
            return "";
        }

        Order order = orderMap.get(currentState).remove(id);
        order.state = State.CANCELED;
        orderMap.get(State.CANCELED).put(id, order);
        return id;
    }

    public String completeOrder(String id) {
        Order order = orderMap.get(State.LIVE).remove(id);
        if (order == null) {
            return "";
        }
        order.state = State.COMPLETED;
        orderMap.get(State.COMPLETED).put(id, order);
        return id;
    }

    public List<String> displayLiveOrders() {
        List<Order> liveOrders = new ArrayList<>(orderMap.get(State.LIVE).values());
        liveOrders.sort(Comparator.comparingLong(o -> o.timestamp));
        List<String> orderIds = new ArrayList<>();
        for (Order order : liveOrders) {
            orderIds.add(order.id);
        }
        return orderIds;
    }

    private Order findOrder(String id) {
        for (Map<String, Order> orders : orderMap.values()) {
            if (orders.containsKey(id)) {
                return orders.get(id);
            }
        }
        return null;
    }

    private State getOrderState(String id) {
        for (State state : State.values()) {
            if (orderMap.get(state).containsKey(id)) {
                return state;
            }
        }
        return null;
    }
}
```
Complexity Analysis:

Time Complexity:

placeOrder, pauseOrder, resumeOrder, cancelOrder, completeOrder: O(1) for each operation due to direct access via hash map.

displayLiveOrders: O(N log N), where N is the number of orders, due to sorting the live orders based on their timestamps.

Space Complexity: O(N)

Follow-up 1
```java
import java.util.*;
import java.math.*;

enum State {
    LIVE, PAUSED, COMPLETED, CANCELED
}

// Order class extended with userId for Follow-up 1
class Order {
    String id;
    String currency;
    int amount;
    int timestamp;       // CHANGED from v1: was long in version 1
    String type;
    State state;
    String userId;       // NEW in Follow-up 1: track which user this order belongs to

    // CHANGED from v1: added userId parameter
    public Order(String id, String currency, int amount, int timestamp, String type, String userId) {
        this.id = id;
        this.currency = currency;
        this.amount = amount;
        this.timestamp = timestamp;
        this.type = type;
        this.state = State.LIVE;
        this.userId = userId; // NEW in Follow-up 1
    }
}

class CryptoTradingSystem {
    // Same as v1: orders grouped by state
    private Map<State, Map<String, Order>> orderMap;

    // NEW in Follow-up 1:
    // Maintain an index from userId to all orders of this user
    private Map<String, List<Order>> userOrderMap;

    public CryptoTradingSystem() {
        // Same initialization logic for orderMap as v1
        orderMap = new EnumMap<>(State.class);
        for (State state : State.values()) {
            orderMap.put(state, new HashMap<>());
        }
        // NEW in Follow-up 1: initialize user -> orders map
        userOrderMap = new HashMap<>();
    }

    // CHANGED from v1: added userId parameter
    public String placeOrder(String id, String currency, int amount, int timestamp, String type, String userId) {
        // Same uniqueness check as v1
        if (findOrder(id) != null) {
            return "";
        }

        // CHANGED from v1: construct Order with userId
        Order order = new Order(id, currency, amount, timestamp, type, userId);
        orderMap.get(State.LIVE).put(id, order);

        // NEW in Follow-up 1:
        // Also record this order in userOrderMap for bulk user operations
        if (!userOrderMap.containsKey(userId)) {
            userOrderMap.put(userId, new ArrayList<>());
        }
        userOrderMap.get(userId).add(order);

        return id;
    }

    // Same logic as v1
    public String pauseOrder(String id) {
        Order order = orderMap.get(State.LIVE).remove(id);
        if (order == null) {
            return "";
        }
        order.state = State.PAUSED;
        orderMap.get(State.PAUSED).put(id, order);
        return id;
    }

    // Same logic as v1
    public String resumeOrder(String id) {
        Order order = orderMap.get(State.PAUSED).remove(id);
        if (order == null) {
            return "";
        }
        order.state = State.LIVE;
        orderMap.get(State.LIVE).put(id, order);
        return id;
    }

    // Same logic as v1
    public String cancelOrder(String id) {
        State currentState = getOrderState(id);
        if (currentState == null || currentState == State.COMPLETED || currentState == State.CANCELED) {
            return "";
        }

        Order order = orderMap.get(currentState).remove(id);
        order.state = State.CANCELED;
        orderMap.get(State.CANCELED).put(id, order);
        return id;
    }

    // Same logic as v1
    public String completeOrder(String id) {
        Order order = orderMap.get(State.LIVE).remove(id);
        if (order == null) {
            return "";
        }
        order.state = State.COMPLETED;
        orderMap.get(State.COMPLETED).put(id, order);
        return id;
    }

    // Essentially same as v1; still sorts live orders by timestamp
    public List<String> displayLiveOrders() {
        List<Order> liveOrders = new ArrayList<>(orderMap.get(State.LIVE).values());
        liveOrders.sort(Comparator.comparingLong(o -> o.timestamp));
        List<String> orderIds = new ArrayList<>();
        for (Order order : liveOrders) {
            orderIds.add(order.id);
        }
        return orderIds;
    }

    // NEW in Follow-up 1:
    // Cancel all cancellable orders (LIVE or PAUSED) for a given userId
    public int cancelAllOrders(String userId) {
        List<Order> orders = userOrderMap.get(userId);
        if (orders == null) {
            return 0;
        }
        int cancelCount = 0;
        // Use a copy of the list to avoid issues if internal state changes
        for (Order order : new ArrayList<>(orders)) {
            String result = cancelOrder(order.id); // reuse single-order cancel logic
            if (!result.isEmpty()) {
                cancelCount++;
            }
        }
        return cancelCount;
    }

    // Same helper as v1
    private Order findOrder(String id) {
        for (Map<String, Order> orders : orderMap.values()) {
            if (orders.containsKey(id)) {
                return orders.get(id);
            }
        }
        return null;
    }

    // Same helper as v1
    private State getOrderState(String id) {
        for (State state : State.values()) {
            if (orderMap.get(state).containsKey(id)) {
                return state;
            }
        }
        return null;
    }
}

```
Complexity Analysis:
Time Complexity:

placeOrder, pauseOrder, resumeOrder, cancelOrder, completeOrder: O(1) for each operation due to direct access via hash map.

displayLiveOrders: O(N log N), where N is the number of orders, due to sorting the live orders based on their timestamps.

cancelAllOrders: O(M), where M is the number of orders associated with the user.

Space Complexity: O(N)


follow-up 2
```java
import java.util.*;
import java.math.*;

enum State {
    LIVE, PAUSED, COMPLETED, CANCELED
}

class Order {
    String id;
    String currency;
    int amount;
    int timestamp;
    String type;
    State state;
    String userId;

    public Order(String id, String currency, int amount, int timestamp, String type, String userId) {
        this.id = id;
        this.currency = currency;
        this.amount = amount;
        this.timestamp = timestamp;
        this.type = type;
        this.state = State.LIVE;
        this.userId = userId;
    }
}


class CryptoTradingSystem {
    // CHANGED from Follow-up 1:
    // Previously: Map<State, Map<String, Order>> orderMap;
    // Now: an extra level keyed by streamId → then state → then orderId
    private Map<String, Map<State, Map<String, Order>>> orderMap; // NEW dimension: streamId

    // CHANGED from Follow-up 1:
    // Previously: Map<String, List<Order>> userOrderMap;
    // Now: streamId → (userId → list of orders)
    private Map<String, Map<String, List<Order>>> userOrderMap;

    // NEW in Follow-up 2: number of streams
    private int streams;

    // CHANGED from Follow-up 1:
    // Previously: CryptoTradingSystem()
    // Now: we take the number of streams as input
    public CryptoTradingSystem(int streams) {
        this.streams = streams;
        orderMap = new HashMap<>();
        userOrderMap = new HashMap<>();

        // NEW in Follow-up 2:
        // Initialize per-stream structures for all states and user maps
        for (int i = 0; i < streams; i++) {
            String streamId = "stream-" + i;
            orderMap.put(streamId, new HashMap<>());
            for (State state : State.values()) {
                orderMap.get(streamId).put(state, new HashMap<>());
            }
            userOrderMap.put(streamId, new HashMap<>());
        }
    }

    // Same signature as Follow-up 1 (with userId), but now we route to a stream
    public String placeOrder(String id, String currency, int amount, int timestamp, String type, String userId) {
        // NEW in Follow-up 2: decide which stream this user belongs to
        String streamId = findUserStream(userId);

        // CHANGED from Follow-up 1:
        // findOrder now also depends on streamId
        if (findOrder(id, streamId) != null) {
            return "";
        }

        Order order = new Order(id, currency, amount, timestamp, type, userId);

        // CHANGED from Follow-up 1:
        // Insert into the LIVE bucket of the chosen stream
        orderMap.get(streamId).get(State.LIVE).put(id, order);

        // CHANGED from Follow-up 1:
        // userOrderMap is now per stream as well
        userOrderMap.get(streamId)
                    .computeIfAbsent(userId, k -> new ArrayList<>())
                    .add(order);

        return id;
    }

    // CHANGED from Follow-up 1:
    // displayLiveOrders now needs to aggregate across all streams
    public List<String> displayLiveOrders() {
        List<Order> liveOrders = new ArrayList<>();

        // NEW in Follow-up 2:
        // Collect LIVE orders from all streams
        for (String streamId : orderMap.keySet()) {
            liveOrders.addAll(orderMap.get(streamId).get(State.LIVE).values());
        }

        liveOrders.sort(Comparator.comparingLong(o -> o.timestamp));
        List<String> orderIds = new ArrayList<>();

        for (Order order : liveOrders) {
            orderIds.add(order.id);
        }
        return orderIds;
    }

    public String pauseOrder(String id) {
        // NEW in Follow-up 2:
        // First figure out which stream contains this order
        String streamId = findOrderStream(id);
        if (streamId.isEmpty()) {
            return "";
        }

        // CHANGED from Follow-up 1: operate within the found stream
        Order order = orderMap.get(streamId).get(State.LIVE).remove(id);
        if (order == null) {
            return "";
        }
        order.state = State.PAUSED;
        orderMap.get(streamId).get(State.PAUSED).put(id, order);
        return id;
    }

    public String resumeOrder(String id) {
        String streamId = findOrderStream(id); // NEW in Follow-up 2
        if (streamId.isEmpty()) {
            return "";
        }

        Order order = orderMap.get(streamId).get(State.PAUSED).remove(id);
        if (order == null) {
            return "";
        }

        order.state = State.LIVE;
        orderMap.get(streamId).get(State.LIVE).put(id, order);
        return id;
    }

    public String cancelOrder(String id) {
        String streamId = findOrderStream(id); // NEW in Follow-up 2
        if (streamId.isEmpty()) {
            return "";
        }

        State currentState = getOrderState(id, streamId); // CHANGED: state lookup is per stream
        if (currentState == null || currentState == State.COMPLETED || currentState == State.CANCELED) {
            return "";
        }

        Order order = orderMap.get(streamId).get(currentState).remove(id);
        if (order == null) {
            return "";
        }

        order.state = State.CANCELED;
        orderMap.get(streamId).get(State.CANCELED).put(id, order);
        return id;
    }

    public String completeOrder(String id) {
        String streamId = findOrderStream(id); // NEW in Follow-up 2
        if (streamId.isEmpty()) {
            return "";
        }

        Order order = orderMap.get(streamId).get(State.LIVE).remove(id);
        if (order == null) {
            return "";
        }

        order.state = State.COMPLETED;
        orderMap.get(streamId).get(State.COMPLETED).put(id, order);
        return id;
    }

    // CHANGED from Follow-up 1:
    // findOrder now only searches within a given stream
    private Order findOrder(String id, String streamId) {
        Map<State, Map<String, Order>> statesMap = orderMap.get(streamId);
        for (State state : State.values()) {
            Order order = statesMap.get(state).get(id);
            if (order != null) {
                return order;
            }
        }
        return null;
    }

    // CHANGED from Follow-up 1:
    // getOrderState now also takes a streamId
    private State getOrderState(String id, String streamId) {
        for (State state : State.values()) {
            if (orderMap.get(streamId).get(state).containsKey(id)) {
                return state;
            }
        }
        return null;
    }

    // NEW in Follow-up 2:
    // Decide which stream a user should go to.
    // This ensures all orders for the same user map to the same stream.
    private String findUserStream(String userId) {
        String[] parts = userId.split("-");
        int num = parts.length > 1 ? Integer.parseInt(parts[1]) : 0;
        num = num % streams;
        return "stream-" + num;
    }

    // NEW in Follow-up 2:
    // Find which stream currently holds the order with this id.
    // This is needed for operations that only know the orderId.
    private String findOrderStream(String id) {
        for (String streamId : orderMap.keySet()) {
            for (State state : State.values()) {
                if (orderMap.get(streamId).get(state).containsKey(id)) {
                    return streamId;
                }
            }
        }
        return "";
    }

    // CHANGED from Follow-up 1:
    // Now cancelAllOrders must first find the correct stream for this user
    public int cancelAllOrders(String userId) {
        String streamId = findUserStream(userId); // NEW: pick the stream from userId
        List<Order> orders = userOrderMap.get(streamId).get(userId);
        if (orders == null) {
            return 0;
        }
        int cancelCount = 0;
        for (Order order : new ArrayList<>(orders)) {
            String result = cancelOrder(order.id); // reuse single-order cancel logic across streams
            if (!result.isEmpty()) {
                cancelCount++;
            }
        }
        return cancelCount;
    }
}
```
Complexity Analysis:
Time Complexity:

placeOrder, pauseOrder, resumeOrder, cancelOrder, completeOrder: O(1) for each operation within a stream due to direct access through mappings.

displayLiveOrders: O(N log N), because it involves sorting the combined live orders from all streams based on their timestamps.

cancelAllOrders: O(M), where M is the number of orders associated with the userId within their designated stream.

Space Complexity: O(N)
