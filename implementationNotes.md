# Deciphering RAPTOR:
(24/05/2022)

<br>

1. Looked at `class McRaptorAlgorithm` ([source code](https://github.com/yunusskeete/pyraptor/blob/42e5303a52e0ce09349fe98fc4968ed38be281b1/pyraptor/model/mcraptor.py#L19))
    1. Looked at implementation of [Bag](https://github.com/yunusskeete/pyraptor/blob/42e5303a52e0ce09349fe98fc4968ed38be281b1/pyraptor/model/mcraptor.py#L32)
    1. Looked at definition of [Bag](https://github.com/yunusskeete/pyraptor/blob/bb43ab268ea08930e829c3c88c92871f951312c3/pyraptor/model/structures.py#L608)
    1. Looked at [pareto_set](https://github.com/yunusskeete/pyraptor/blob/bb43ab268ea08930e829c3c88c92871f951312c3/pyraptor/model/structures.py#L631)
    1. Looked at definition of [pareto_set](https://github.com/yunusskeete/pyraptor/blob/bb43ab268ea08930e829c3c88c92871f951312c3/pyraptor/model/structures.py#L776)
    1. Looked at [Label](https://github.com/yunusskeete/pyraptor/blob/bb43ab268ea08930e829c3c88c92871f951312c3/pyraptor/model/structures.py#L786)
    1. Looked at definition of [Label](https://github.com/yunusskeete/pyraptor/blob/bb43ab268ea08930e829c3c88c92871f951312c3/pyraptor/model/structures.py#L561)


[Label source code:](https://github.com/yunusskeete/pyraptor/blob/bb43ab268ea08930e829c3c88c92871f951312c3/pyraptor/model/structures.py#L561)

```python
@dataclass(frozen=True)
class Label:
    """Label"""

    earliest_arrival_time: int
    fare: int  # total fare
    trip: Trip  # trip to take to obtain travel_time and fare
    from_stop: Stop  # stop to hop-on the trip
    n_trips: int = 0
    infinite: bool = False

    @property
    def criteria(self):
        """Criteria"""
        return [self.earliest_arrival_time, self.fare, self.n_trips]

    def update(self, earliest_arrival_time=None, fare_addition=None, from_stop=None):
        """Update earliest arrival time and add fare_addition to fare"""
        return copy(
            Label(
                earliest_arrival_time=earliest_arrival_time
                if earliest_arrival_time is not None
                else self.earliest_arrival_time,
                fare=self.fare + fare_addition
                if fare_addition is not None
                else self.fare,
                trip=self.trip,
                from_stop=from_stop if from_stop is not None else self.from_stop,
                n_trips=self.n_trips,
                infinite=self.infinite,
            )
        )

    def update_trip(self, trip: Trip, current_stop: Stop):
        """Update trip"""
        return copy(
            Label(
                earliest_arrival_time=self.earliest_arrival_time,
                fare=self.fare,
                trip=trip,
                from_stop=current_stop if self.trip != trip else self.from_stop,
                n_trips=self.n_trips + 1 if self.trip != trip else self.n_trips,
                infinite=self.infinite,
            )
        )
```

- We could add an occupancy attribute to the class [Label](https://github.com/yunusskeete/pyraptor/blob/bb43ab268ea08930e829c3c88c92871f951312c3/pyraptor/model/structures.py#L561), which contains the (int) occupancy at stop `from_stop` (`p`) <br> (see commented implementation - 25/05/2022)
- We could add an occupancy attribute to the class [Trip](https://github.com/yunusskeete/pyraptor/blob/42e5303a52e0ce09349fe98fc4968ed38be281b1/pyraptor/model/structures.py#L246), which contains the (int) occupancy on trip, `trip` (`t(l)`)
- In the `criteria` attribute of the class [Label](https://github.com/yunusskeete/pyraptor/blob/bb43ab268ea08930e829c3c88c92871f951312c3/pyraptor/model/structures.py#L561), we can either return a weighted combination of stop and trip occupancy (weighted by time spent?) or return both as separate criteria.


To Do:
1. (26/05/2022) Read through and understand all of the structures in [structures](/pyraptor/model/structures.py)
1. (26/05/2022) Run a few raptor queries and display these structures as debug outputs
1. (26/05/2022) Understand the FULL code workflow to go from query to route

Then:
1. Begin by creating a new branch in the [GitHub repository](https://github.com/yunusskeete/pyraptor) and implementing these new class attributes (assign all to `int: 0`)
1. Ensure that these are handled correctly by [McRaptorAlgorithm](https://github.com/yunusskeete/pyraptor/blob/42e5303a52e0ce09349fe98fc4968ed38be281b1/pyraptor/model/mcraptor.py#L19), [Bag](https://github.com/yunusskeete/pyraptor/blob/bb43ab268ea08930e829c3c88c92871f951312c3/pyraptor/model/structures.py#L608), [pareto_set](https://github.com/yunusskeete/pyraptor/blob/bb43ab268ea08930e829c3c88c92871f951312c3/pyraptor/model/structures.py#L776) and [Label](https://github.com/yunusskeete/pyraptor/blob/bb43ab268ea08930e829c3c88c92871f951312c3/pyraptor/model/structures.py#L561).
1. Get accurate values for occupancy (stop) and occupancy (trip) from database ()Extend the [Timetable](https://github.com/yunusskeete/pyraptor/blob/42e5303a52e0ce09349fe98fc4968ed38be281b1/pyraptor/model/structures.py#L24) class.

<br>

<br>

<br>

<br>

<br>

---

<br>

<br>

<br>

<br>

<br>

# Occupancy
(25/05/2022)


To add occupancy to the rMcRaptor, there are two options:
1. We update the `Timetable` class which is inherited into all Raptor instances. (See below).

    ```python
    @dataclass
    class Timetable:
        """Timetable data"""

        stations: Stations = None
        stops: Stops = None
        trips: Trips = None
        trip_stop_times: TripStopTimes = None
        routes: Routes = None
        transfers: Transfers = None

        def counts(self) -> None:
            """Print timetable counts"""
            logger.debug("Counts:")
            logger.debug("Stations   : {}", len(self.stations))
            logger.debug("Routes     : {}", len(self.routes))
            logger.debug("Trips      : {}", len(self.trips))
            logger.debug("Stops      : {}", len(self.stops))
            logger.debug("Stop Times : {}", len(self.trip_stop_times))
            logger.debug("Transfers  : {}", len(self.transfers))
    ```

    - (See Github [pyraptor/model/structures.py](https://github.com/yunusskeete/pyraptor/blob/bb43ab268ea08930e829c3c88c92871f951312c3/pyraptor/model/structures.py#L24) for `Timetable` class implementation).
    - To achieve this, we will need to implement a `occupancy_stations` class and an `occupancy_trips` class.
    - These classes will need to be live updating or previously specified and from which, you should be able to search for the 'real-time' occupancy corresponding to a platform, or trip (could also include interchange, station etc.).
    <!-- - Using the `get_fare()` class ([definition](https://github.com/yunusskeete/pyraptor/blob/bb43ab268ea08930e829c3c88c92871f951312c3/pyraptor/model/structures.py#L294), [implementation](https://github.com/yunusskeete/pyraptor/blob/bb43ab268ea08930e829c3c88c92871f951312c3/pyraptor/model/mcraptor.py#L138)), we should be able to implement occupancy depreferencing by adding a fare. -->

1. we have a database look-up/update for each trip/stop?

<br>

<br>

<br>

---

<br>

<br>

<br>

# Implementing Occupancy:

**First**, I will just add a fixed value of occupancy for each leg of 0.

- I have updated the `Label` class ([here](pyraptor/model/structures.py)) to include occupancy:

    1. Labels are initialised with `occupancy: int = 0`:
        ```python
        occupancy: int = 0
        ```

    1. The `Label.Criteria` method returns `occupancy`:
        ```python
        return [self.earliest_arrival_time, self.fare, self.n_trips, self.occupancy]
        ```

    1. The `Label.update` method adds the variable `occupancy_addition` to `self.occupancy`:
        ```python
        occupancy=self.occupancy + occupancy_addition
        if occupancy_addition is not None
        else self.occupancy,
        ```
        - TO-DO: Understand use of `fare_addition` for inspiration for implementation of `occupancy_addition`.

    1. The `Label.update_trip` method returns a copy of the `Label` (including `occupancy`):
        ```python

        Label(
            earliest_arrival_time=self.earliest_arrival_time,
            fare=self.fare,
            trip=trip,
            from_stop=current_stop if self.trip != trip else self.from_stop,
            occupancy=self.occupancy, # just occupancy, not with the occupancy_addition? Handled in the Label.update method (I believe)
            n_trips=self.n_trips + 1 if self.trip != trip else self.n_trips,
            infinite=self.infinite,
        )
        ```

<!-- Occupancy in class [Leg](pyraptor/model/structures.py) (structures) and  method [best_legs_to_destination_station](RAPTOR.ipynb) (query_mcraptor) -->

- I have updated the `Leg` class ([here](pyraptor/model/structures.py)) to include occupancy:

    1. Legs are initialised with `occupancy: int = 0`:
        ```python
        occupancy: int = 0
        ```

    1. The `Leg.Criteria` method returns `occupancy`:
        ```python
        return [self.earliest_arrival_time, self.fare, self.n_trips, self.occupancy]
        ```

    1. (TO BE IMPLEMENTED) The property `Leg.occ` finds the occupancy from `self.trip.stop_times: TripStopTime`:
        ```python
        @property
        def occ(self):
            """Arrival time"""
            return [ # tst = TripStopTime
                tst.occupancy for tst in self.trip.stop_times if self.to_stop == tst.stop
            ][0]
        ```
        <!-- - **TO DO:** Add property `occupancy` to `TripStopTime` -->

    1. The `Leg.to_dict` method returns a dictionary including `self.occupancy`:
        ```python

        def to_dict(self, leg_index: int = None) -> Dict:
            """Leg to readable dictionary"""
            return dict(
                trip_leg_idx=leg_index,
                departure_time=self.dep,
                arrival_time=self.arr,
                from_stop=self.from_stop.name,
                from_station=self.from_stop.station.name,
                to_stop=self.to_stop.name,
                to_station=self.to_stop.station.name,
                trip_hint=self.trip.hint,
                trip_long_name=self.trip.long_name,
                from_platform_code=self.from_stop.platform_code,
                to_platform_code=self.to_stop.platform_code,
                fare=self.fare,
                occupancy=self.occupancy,
            )
        ```

-  I have updated the `Trip` class ([here](pyraptor/model/structures.py)) to include occupancy:

    1. (TO BE IMPLEMENTED) The `Trip.get_stop_occupancy` method returns `self.get_stop(depart_stop).occupancy`:
        ```python
        ### LOOK AT STOP CLASS FIRST
        def get_stop_occupancy(self, depart_stop: Stop) -> int: # 03/06/2022
            """Get stop occupancy from depart_stop"""
            stop_occupancy: TripStopTime = self.get_stop(depart_stop)
            return 0 if stop_occupancy is None else stop_occupancy.occupancy
        ```
        <!-- - **TO DO:** Add property `occupancy` to `TripStopTime` -->

    1. The `Leg.to_dict` method returns a dictionary including `self.occupancy`:
        ```python

        def to_dict(self, leg_index: int = None) -> Dict:
            """Leg to readable dictionary"""
            return dict(
                trip_leg_idx=leg_index,
                departure_time=self.dep,
                arrival_time=self.arr,
                from_stop=self.from_stop.name,
                from_station=self.from_stop.station.name,
                to_stop=self.to_stop.name,
                to_station=self.to_stop.station.name,
                trip_hint=self.trip.hint,
                trip_long_name=self.trip.long_name,
                from_platform_code=self.from_stop.platform_code,
                to_platform_code=self.to_stop.platform_code,
                fare=self.fare,
                occupancy=self.occupancy,
            )
        ```

- I have updated the `TripStopTime` class ([here](pyraptor/model/structures.py)) to include occupancy:

    1. The `TripStopTime` method is initialised with `occupancy`:

        ```python
            occupancy = attr.ib(default=0) # EXAMPLE: 0
        ```

        - **IN FUTURE** This is where we do a database look-up to include dynamic `occupancy`.

            - `TripStopTime` classes include `trip`, 'stop`, `dts_arr`, `dts_dep`, `fare`, and now `occupancy`.
            - We can calculate `occupancy` = `num_people` * (`dts_dep` - `dts_arr`)