```java
/*
 * Click `Run` to execute the snippet below!
 */

import java.io.*;
import java.util.*;


class Solution {
    PriorityQueue<Integer> buyOrdersHeap;
    PriorityQueue<Integer> sellOrdersHeap;

    public Solution(int[] buyOrders, int[] sellOrders) {
        buyOrdersHeap = new PriorityQueue<>(Collections.reverseOrder());
        sellOrdersHeap = new PriorityQueue<>();

        for (int orderPrice : buyOrders) {
            buyOrdersHeap.offer(orderPrice);
        }

        for (int orderPrice : sellOrders) {
            sellOrdersHeap.offer(orderPrice);
        }

    }

    public double addOrder(int price, String orderType) {
        if (orderType.equals("sell")) {
            sellOrdersHeap.offer(price);
        } else if (orderType.equals("buy")) {
            buyOrdersHeap.offer(price);
        }

        if (buyOrdersHeap.size() > 0 && sellOrdersHeap.size() > 0) {
            if (buyOrdersHeap.peek() >= sellOrdersHeap.peek()) {
                int sellPrice = sellOrdersHeap.poll();
                int buyPrice = buyOrdersHeap.poll();
                return (sellPrice + buyPrice) / 2.0;
            }
        }
        return -1;
     }
  public static void main(String[] args) {
    int[] buyOrders = {100,100,98,99,97};
    int[] sellOrders = {109,110,110,114,115,119};
    Solution sol = new Solution(buyOrders, sellOrders);
    // [200,"buy"],[105,"buy"],[100,"sell"],[130,"sell"]]
    double t1 = sol.addOrder(200, "buy");
    double t2 = sol.addOrder(105, "buy");
    double t3 = sol.addOrder(100, "sell");
    double t4 = sol.addOrder(130, "sell");

    System.out.println("t1 = " + t1);
    System.out.println("t2 = " + t2);
    System.out.println("t3 = " + t3);
    System.out.println("t4 = " + t4);
    
  }
}

```
