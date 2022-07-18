# myNotes

## Contents:
1. [Understanding Raptor](#1-understanding-raptor)
2. [Understanding McRAPTOR](#2-understanding-mcraptor)
2. [Understanding phiRAPTOR](#3-understanding-phiraptor)
<!-- <div id='understandingRAPTOR'/> -->

## 1. Understanding RAPTOR
RAPTOR works on a timetable ($\Pi$, $\mathcal{S}$ , $\mathcal{T}$  , $\mathcal{R}$ , $\mathcal{F}$), extracted from GTFS stoptimes, stops, trips, routes and transfers respectively, where $\Pi$ $\subset$ $\mathbb{N}_{0}$ is the period of operation (seconds in a day); $\mathcal{S}$ is a set of stops, $\mathcal{T}$ a set of trips, $\mathcal{R}$ a set of routes, and $\mathcal{F}$ a set of transfers or foot-paths.
Each stop, $s$ $\in$ $\mathcal{S}$, corresponds to a boarding/alighting location.
Each trip $t$ $\in$ $\mathcal{T}$ represents a sequence of stops a specific public transit vessel visits along a line.
<!-- % At each stop in the sequence, it may drop off or pick up passengers. -->
For each stop, $s$ $\in$ $\mathcal{S}$, a trip $t$ has associated arrival and departure times $\tau_{arr}(t, p)$, $\tau_{dep}(t, p)$ $\in$ $\Pi$, with $\tau_{arr}(t, p)$ $\le$ $\tau_{dep}(t, p)$.
<!-- % The first and last stops of a trip have an undefined arrival and departure time, respectively. -->
Each route, $r$ $\in$ $\mathcal{R}$, consists of the trips, $t$ $\in$ $\mathcal{T}$, that share the same sequence of stops: there are many more trips than routes ($\mathcal{T}$ $>$ $\mathcal{R}$).
Finally, footpaths, $f$ $\in$ $\mathcal{F}$ model walking connections, or transfers, between stops.

Each transfer consists of two stops $p_{1}$ and $p_{2}$ with an associated constant walking time $\ell$($p_{1}$, $p_{2}$).
<!-- % Note that $\mathcal{F}$ is transitive: if $p_{1}$ and $p_{2}$ are indirectly connected by foot-paths, u is contained in $\mathcal{F}$ as well. -->
The output produced by the RAPTOR algorithm on a timetable is a set of journeys $\mathcal{J}$; a sequence of trips and foot-paths in the order of travel.
Unlike other graph-based algorithms, it is upon this timetable that shortest path queries are made.

```
LAYMAN'S TERMS:
-

1. GTFS is a store of the stoptimes, stops, trips, routes and transfers associated with a transit network in csv format.

2. Rather than building a network graph to run shortest path searches, RAPTOR implements a timetable.

3. A timetable class is made which processes the GTFS data to make stoptimes, stops, trips, routes and transfers RELATIONAL.
    i.      A journey satisfies the shortest path search between an origin and destination station, and details the trips (connections between successive stations) that follow (snippets of) routes (a timetabled service running along a specified line) and the transfers (platform changes) to take one from origin to destination in a sequential manner.
    ii.     Trips specify from_station and to_station and have an associated start_time and end_time with them.
    (Note: there is a unique trip for each train that runs from station A to station B throughout the day. In actuality, from_station and to_station specifies platforms at their parent station rather than the station itself.)
    iii.    Stations have associated with them platforms, from which trips originate or terminate.
    iv.     Transfers have associated with them a from_platform and to_platform and a CONSTANT tranfer time.

4. Hence, the RAPTOR timetable enables one to traverse a route and see, associated with that route, the trips, stops and stoptimes that comprise it - as is implemented within the RAPTOR algorithm as a key part of the shortest path search.
```

Let $p_{s} \in S$ be the source stop, $p_{t} \in S$ be the target stop, and $\tau \in \Pi$ be the departure time.
The shortest path between $p_{s}$ and $p_{t}$ poses a bicriteria problem.
RAPTOR solves the bicriteria problem minimizing arrival time and number of transfers.
We say a journey $J_{1} dominates a journey $J_{2}$, denoted by $J_{1} \le J_{2}$, if $J_{1} is no worse in any criterion than $J_{2}$.
RAPTOR operates in rounds, $k$, and computes for every $k$ a nondominated journey to a target stop $p_{t}$ with minimum arrival time having at most $k$ trips.
Round $k$ computes the fastest way of getting to every stop with at most
$k$ − 1 transfers (i. e., by taking at most $k$ trips).
Note that some stops may not be reachable at all.
The number of rounds is bound by $K$ such that $k \in K$.
The algorithm associates with each stop $p$ a multilabel ($\tau_{0}(p)$, $\tau_{1}(p)$, ... , $\tau_{K}(p)$), where $\tau_{i}(p)$ represents the earliest known arrival time at $p$ with up to $i$ trips.
All values in all labels are initialized to $\infty$.
We then set $\tau_{0}(p_{s}) = \tau$.
We maintain the following invariant:
at the beginning of round $k$ (for $k \ge 1$), the first $k$ entries in $\tau(p)$ (from $\tau_{0}(p)$ to $\tau_{k-1}(p)$) are correct, i.e., entry $\tau_{i}(p)$ represents the earliest arrival time at $p$ using at most $i$ trips.
The remaining entries are set to $\infty$.
The goal of round $k$ is to compute $\tau_{k}(p)$ for all $p$.
It does so in three stages:

The first stage of round $k$ sets $\tau_{k}(p) = \tau_{k-1}(p)$ for all stops $p$:
$\tau_{k}(p)$ will only be updated if it can be improved in this round, hence seting an upper bound on the earliest arrival time at $p$ with at most $k$ trips as the earliest arrival time from the previous round.

The second stage then processes each route in the timetable exactly once.
Consider a route $r$, and let $\mathcal{T}(r) = (t_{0}, t_{1}, ... , t_{|\mathcal{T}(r)|−1})$ be the sequence of trips that follow route SrS, from earliest to latest.
When processing route $r$, we consider journeys where the last ($k^{th}$) trip taken is in route $r$.
Let $et(r, p_{i})$ be the earliest trip in route r that one can catch at stop $p_{i}$, i.e., the earliest trip $t$ such that $\tau_{dep}(t, p_{i}) \le \tau_{k−1}(p_{i})$.
(Note that this trip may not exist, in which case $et(r, p_{i})$ is undefined.)
To process the route, we visit its stops in order until we find a stop $p_{i}$ such that $et(r, p_{i})$ is defined.
This is when we can “hop on” the route.
Let the corresponding trip $t$ be the current trip for $k$.
We keep traversing the route.
For each subsequent stop $p_{j}$, we can update $\tau_{k}(p_{j})$ using this trip.

To reconstruct the journey to a stop, $p$, we set a parent pointer to the stop at which $t$ was boarded.
Moreover, we may need to update the current trip for $k$:
at each stop $p_{i}$ along $r$ it may be possible to catch an earlier trip (because a quicker path to $p_{i}$ has been found in a previous round).
Thus, we have to check if $\tau_{k−1}(p_{i}) < \tau_{arr}(t, p_{i})$ and update $t$ by recomputing $et(r, p_{i})$.

Once we have reconstructed the journey, we have found the earliest arrival time to each stop in our network, the trips and transfers we took to get there and a parent pointer to where we boarded this trip.
Concretely, we have found $et(r, p)$ for each route, $r \in \mathcal{R}$ and stop, $p \in \mathcal{S}$.
Recall that associated with each trip are the attributes from_stop, to_stop, from_time, to_time.
Hence, we have found the stoptimes, stops, trips, routes and transfers to satisfy this shortest path query.

For improvements, see RAPTOR paper.
```
LAYMAN'S TERMS:
-

1. We start at a source stop, p_{s}, and departure time, \tau, and intend to find the shortes path to the target stop, p_{t}.
    i.      We do this by finding the earliest arrival with least transfers to ALL stations by traversing all possible routes sequentially (in rounds, k).

2. We startwith round 1 (k = 1). Hence, in this round, we will find the earliest arrival time to every station reachable from p_{s} with at most k - 1 (=0) transfers.

3. We assume that the earliest arrival time to all other stations on the network is infinite.

4. We traverse all of the routes from p_{s} that depart after \tau, noting the trips that we take to get there.

5. As we pass a stop on this traversal, we update their earliest arrival time with the arrival time for this trip, IF it represents an improvement on the current earliest arrival time.
    i.      Note that in round 1, all earliest arrival times are initialised to infinity, so this criteria is always met.

6. We continue until we have fully traversed each route, marking the earliest arrival times at the stops visited.

7. We proceed to the next round (k=2), hence, we find the earliest arrival time at all stations allowing for up to k - 1 (=1) transfers.
    i.      To do this, we continue from every stop we reached in the previous round.
    ii.     We assess if there are any transfers we can make to traverse another route.
    iii.    If we can traverse another route, we do so, again noting down the earliest arrival time to the stops it serves, the trip taken to get there, AND the stop at which we boardd this trip IF it represents an improvement (on initialised value, infinity, or the updated value from the previous round).
    iv.     Once we have traversed all of the routes accessible in this round, the round is complete.

8. We repeat this process for rounds k <= K. Hence, at round K, we yield the earliest possible arrival time using at most K trips, the trip we used to get there, and the stop at which we boarded this trip.

9. To reconstruct our journey to our destination station, we lok at its earliest arrival time, the trip we used to get there (which details trip times), and the stop at which we boarded this trip.
    i.      The stop at which we boarded the trip to our destination will contain an earliest arrival time, the trip we used to get there, and the stop at which we boarded this trip.
    ii.     We work backwards, looking through the various "legs" of this journey until we arrive back at our origin station, p_{s}.
    iii.    This details our route and solves the shortest path problem.
```

RAPTOR scans routes in no particular order, and thus, is highly parallelisable.

<br>

<br>

## 2. Understanding McRAPTOR

Recall that plain RAPTOR stores exactly one value $\tau_{k}(p)$ per stop
and round.
To extend the algorithm to more criteria, we keep multiple nondominating labels for each stop $p$ in round $k$.
We store these labels in bags, denoted by $B_{k}(p)$.
The algorithm is then modified as follows.
When relaxing a route $r$, we first create an empty route bag $B_{r}$.
Each label $L$ in the route bag has an associated active trip $t(L)$.
When traversing the stops of $r$ in order, we process each stop $p$ in three steps.
The first step updates the arrival times of every label $L \in B_{r}$ according to their associated trips $t(L)$ at $p$.
<!-- Note that if two labels have the same associated trip, one might be eliminated. -->
In the second step, we merge $B_{r}$ into $B_{k}(p)$ by copying all labels from $B_{r}$ to $B_{k}(p)$, and discarding dominated labels in $B_{k}(p)$. 
The final step merges $B_{k−1}(p)$ into $B_{r}$ and assigns trips to all newly
added labels.

Like RAPTOR, McRAPTOR scans routes in no particular order, and thus, can be parallelized in the same way.


<br>

<br>


## 3. Understanding phiRAPTOR

PhiRAPTOR introduces the additional criteria of `occupancy` and `COVID_risk` to the pre-existing criteria of earliest arrival time, number of trips and fare.
occupancy represents a trip-duration-weighted sum of the expected occupancy on each trip.
Expected trip occupancies are found using either the current or forecasted occupancy depending on whether trips occur now or in the future.
In implementation, COVID_risk is always the value at the time of journey query (departure time).
Currently, we neglect the consideration of occupancy for transfers (do we?).