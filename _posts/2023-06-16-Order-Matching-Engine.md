---
layout: post
title: "Creating a simple Order Matching Engine (OME)"
categories: post
---

In this, I am writing about how I went about creating a simple OME. Disclaimer: no networking related stuff, including FIX, was used in this.  For the curren version on GitHub, I have not used any sophisticated libraries like Boost, although it is defnitely something I am curious to learn and experiment with in future versions (if any).

So let's start with the different components. First is market tick data, second is a way to feed this data into the OME and third is the OME containing the orderbook. One could use an API like interface that reads order and calls the appropriate API to handle the request. However, for this simulation, I have simplifies it to just reading a list of orders from a CSV. The CSV contains orders in a prespecified format. The allowed interactions are add order, cancel order and print order book for now.


## Implementation details

Now let's define the classes or objects we would need:<br>
* Orders
* Limit
* OrderBook

Orders class is a representation of an individual order. This could contain various atttributes such as timestamps, size, type, etc, which we will list in detail soon. Some of the attributes here are quite self explanatory, We maintain the number of stocks filled out of the total order size in case there are only partial fills. I define type (ADD, CANCEL) and order status (PENDING, DONE, CANCELLED) as enums for most efficient matching when writing switch cases. We use switch cases quite a lot as they are much more efficient than if-else. Addtionally, comparison of enums in switch is much faster compared to string or other types. We also maintain the next and prev pointers indicating the next and previous order in the limit queue respectively. We will also define some methods on those orders such as `cancel` or `fill` order.

    int id;
    int limitPrice;
    int size; // what was the original size of the order placed
    int filled { 0 }; // how much of the order has currently been filled

    types type;
    states status { PENDING };

    std::chrono::system_clock::time_point placedDate; // time at which the order was added to the queue
    std::chrono::system_clock::time_point actionDate; // time at which the order was removed (cancelled or executed) from the queue

    Order* next { nullptr };
    Order* prev { nullptr };

The Limit object is a container for the queue (linked list) of orders in sequence of priority at a particular limit price, hence it is essentially characterized by a limit price. We define the head and tail of the linked list using the first two pointers where the head is a pointer to the order object instance which has the highest priority (gets filled first) in the limit queue. We also keep a track of the total number of orders and total volume across al orders in the limit which will be useful when printing the order book. We use a linked list for the limit queue and not an actual queue data structure as the linked list allows for O(1) cancel/update. So given an order ID, we can locate the order's position in the linked list and udpate the linked list in O(1), whereas in a queue, only the first and last element are accessible in O(1). 

    Order* head { nullptr };
    Order* tail { nullptr };

    int price;
    int volume { 0 };
    int numberOfOrders { 0 };

Lastly, the Orderbook object is the object containing all the limit objects at their respective limit price in an array and pointers to the best buy and best sell offer limit object. Here we want the lookup and moving to the next best price limit object after fulfilling the current best limit to be as close to O(1) as possible. We could have a tree of limit objects in order of limit price for this. This is particularly useful when the order book is illiquid and the next best buy/sell order can be many many decimals awat from the current best, since we would directly have the next best order as the children of the root. However, if the order book is liquid enough, even using an array (with indexes denoting limit price*100). Arrays are much more space and time efficient comapred to trees and if the next best orders lies within 2-3 decimal points (liquid prder book), the orderbook can be updated in O(1) when an order is matched/placed.

    Limit* buySide[100000] { nullptr }; 
    Limit* sellSide[100000] { nullptr };

    int numberOfSellOrders = 0;
    int numberOfBuyOrders = 0;

    Limit* bestSell {nullptr};
    Limit* bestBuy {nullptr};


After this, most things should be self explanatory. You can define methods for cancel and adding order. For this to be more compact and understandable, I iterated over several versions and came up with a final implementation where I felt there was most resusability of code. For instance, when you cancel an order, the orderbook provides the interface `cancel order` function, which udpated best Sell/buy and the array, we then call the `remove order from queue` fucntion which is a method of the limit object, we then call the cancel order fucntion within the Order object which changes the status and other attributes of the order such as action date and uupdates the next, prev pointers to null.

## Matching Engine

The OME accepts only limit orders for now, but can easily be extended to market orders, which we will be attempting in the upcoming verions. The logic is simply this: suppose there is a buy limit order is placed at price `p` and the best sell is at `bs`. If `p` > `bs`, match all sell orders starting from `bs` up until `p` (inclusive) in the order of increasing limit price till the quantity of the buy order at `p` is satisfied. If the existing sell order until price `p` are not able to completely fulfill the incoming buy order, the best buy updates to `p` and the best sell udpates to `p + 0.01`. Essentially, limit order in buy side means match all sell orders lower or equal to the limit price of the buy order, if any (when `p` < `bs`).  


## Simulating the market feed

So I couldn't find a market simulator, I build one. I used pesudo random generators for simulating a market around a fixed price. So I uniformly pick price from a range with fixed mean and distance, and arbitrary order size and order type. This is very simplified but does the job of testing your order matching engine and fills the order book suffciciently. I store these order in a CSV and then read the CSV sequentially. As an extension, I am planning  to build a better simulator for market feed using moving average pricing and more involved distrbution to pick prices from.

## Output 

The logs of all add, match and cancel orders are logged into a log file with a time stamp with microsecond granularity. The output when there is a `print order book` call looks like this:


                                    ________ Sell ________ 

                                 Price      #Orders    Volume         
                                 79.64         2          142        
                                 79.63         1          117        
                                 79.62         3          600        
                                 79.53         1          366        
                                 79.52         1          25         
                                 79.48         1          426        
                                 79.41         2          580        
                                 79.32         1          210        
                                 79.28         1          450        
                                 79.25         2          331        
                                 79.20         1          338        
                                 79.19         1          20         
                                 79.00         1          459        
                                 78.81         2          692        
                                 78.75         1          155        

    _________ Buy _________ 

    Price      #Orders    Volume     
    78.65         1          131        
    78.55         2          160        
    78.54         1          325        
    78.53         1          132        
    78.50         1          29         
    78.49         2          735        
    78.47         1          96         

### References:
* [https://www.cmegroup.com/confluence/display/EPICSANDBOX/CME+Globex+Matching+Algorithm+Steps#CMEGlobexMatchingAlgorithmSteps-TOP](https://www.cmegroup.com/confluence/display/EPICSANDBOX/CME+Globex+Matching+Algorithm+Steps#CMEGlobexMatchingAlgorithmSteps-TOP)
* [https://quant.stackexchange.com/questions/3783/what-is-an-efficient-data-structure-to-model-order-book](https://quant.stackexchange.com/questions/3783/what-is-an-efficient-data-structure-to-model-order-book)
* [https://web.archive.org/web/20110219163448/http://howtohft.wordpress.com/2011/02/15/how-to-build-a-fast-limit-order-book/](https://web.archive.org/web/20110219163448/http://howtohft.wordpress.com/2011/02/15/how-to-build-a-fast-limit-order-book/)

PS: I write these posts for my own reference and not as a reference for people trying to learn more about the topic. 