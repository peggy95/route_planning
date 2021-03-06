# Robust Journey Planning

## Problem Description

In this final project we will build your own public transport route planner to improve on that. We will reuse the SBB dataset (See next section: [Dataset Description](#dataset-description)).

Given a desired departure, or arrival time, route planner will compute the fastest route between two stops within a provided uncertainty tolerance expressed as interquartiles. For instance, "what route from A to B is the fastest at least Q% of the time if I want to leave from A (resp. arrive at B) at instant T". Note that *uncertainty* is a measure of a route not being feasible within the time computed by the algorithm.

we will do:
- Model the public transport infrastructure for your route planning algorithm.
- Build a predictive model using historical arrival/departure time data and potentially other sources of data for your public transport network.
- Implement a robust route planning algorithm using this predictive model.
- Implement a method to test and validate your results.
- Implement a web visualization to demonstrate your method.

Solving this problem accurately can be difficult. We used a few simplifying assumptions:

- There is no penalty for assuming that delays or travel times on the public transport network are uncorrelated with one another. You will get extra credits if you do.
- Once a route is computed, a traveller is expected to follow the planned routes to the end, or until it fails with unknown consequences (e.g. miss a connection). You **do not** need to address the case where travellers are able to defer their decisions and adapt their journey "en route", as more information becomes available. This requires to consider all alternative routes (contingency plans) in the computation of the uncertainty levels, which is more difficult to implement.
- The planner will not need to mitigate the traveller's inconvenience if a plan fails. Two routes with identical travel times under the uncertainty tolerance are equivalent, even if outside this uncertainty tolerance on one route has a much worse outcome than the other.

## Dataset Description

For this project we will use the data published by the Open Data Platform Swiss Public Transport (<https://opentransportdata.swiss>).

- You can download it using the following links.
    - <https://os.unil.cloud.switch.ch/swift/v1/CFF/2017-09.tar.lzma>
    - <https://os.unil.cloud.switch.ch/swift/v1/CFF/2017-10.tar.lzma>
    - <https://os.unil.cloud.switch.ch/swift/v1/CFF/2017-11.tar.lzma>
    - <https://os.unil.cloud.switch.ch/swift/v1/CFF/2017-12.tar.lzma>
    - <https://os.unil.cloud.switch.ch/swift/v1/CFF/2018-01.tar.lzma>
    - <https://os.unil.cloud.switch.ch/swift/v1/CFF/2018-02.tar.lzma>
    - <https://os.unil.cloud.switch.ch/swift/v1/CFF/2018-03.tar.lzma>
    - <https://os.unil.cloud.switch.ch/swift/v1/CFF/2018-04.tar.lzma>
    - <https://os.unil.cloud.switch.ch/swift/v1/CFF/metadata.tar.gz>

The folder contains the actual data [istdaten](<https://opentransportdata.swiss/en/dataset/istdaten>) and the station list data [BFKOORD_GEO](https://opentransportdata.swiss/de/cookbook/hafas-rohdaten-format-hrdf/#Abgrenzung).

Format: the dataset is presented a collection of textfiles with fields separated by ';' (semi-colon). There is one file per day.

Unfortunately, the full description from opentransportdata.swiss is only provided in German. You can use an automated translator (we recommend [DeepL](<https://www.deepl.com>)) to get more information, but here are the relevant column descriptions:

- `BETRIEBSTAG`: date of the trip
- `FAHRT_BEZEICHNER`: identifies the trip
- `BETREIBER_ABK`, `BETREIBER_NAME`: operator (name will contain the full name, e.g. Schweizerische Bundesbahnen for SBB)
- `PRODUCT_ID`: type of transport, e.g. train, bus
- `LINIEN_ID`: for trains, this is the train number
- `LINIEN_TEXT`,`VERKEHRSMITTEL_TEXT`: for trains, the service type (IC, IR, RE, etc.)
- `ZUSATZFAHRT_TF`: boolean, true if this is an additional trip (not part of the regular schedule)
- `FAELLT_AUS_TF`: boolean, true if this trip failed (cancelled or not completed)
- `HALTESTELLEN_NAME`: name of the stop
- `ANKUNFTSZEIT`: arrival time at the stop according to schedule
- `AN_PROGNOSE`: actual arrival time (when `AN_PROGNOSE_STATUS` is `GESCHAETZT`)
- `AN_PROGNOSE_STATUS`: look only at lines when this is `GESCHAETZT`. This indicates that `AN_PROGNOSE` is the measured time of arrival.
- `ABFAHRTSZEIT`: departure time at the stop according to schedule
- `AB_PROGNOSE`: actual departure time (when `AN_PROGNOSE_STATUS` is `GESCHAETZT`)
- `AB_PROGNOSE_STATUS`: look only at lines when this is `GESCHAETZT`. This indicates that `AB_PROGNOSE` is the measured time of arrival.
- `DURCHFAHRT_TF`: boolean, true if the transport does not stop there

Each line of the file represents a stop and contains arrival and departure times. When the stop is the start or end of a journey, the corresponding columns will be empty (`ANKUNFTSZEIT`/`ABFAHRTSZEIT`).
In some cases, the actual times were not measured so the `AN_PROGNOSE_STATUS`/`AB_PROGNOSE_STATUS` will be empty or set to `PROGNOSE` and `AN_PROGNOSE`/`AB_PROGNOSE` will be empty.

We will use the SBB data limited around the Zurich area. We will focus on all the stops within 10km of the Zurich train station.


