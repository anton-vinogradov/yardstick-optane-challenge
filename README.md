Just copy repository content to apache-ignite-X.Y.Z-bin/benchmarks/config/ folder and start one of the following benchmarks:
- inmem (Inmemory mode, no discs used)
- optane1-pds-optane1-wal (Single Optane used)
- optane1-pds-optane2-wal (One Optane used to store persistence and another to store WAL)
- ssd1-pds-ssd1-wal (Single SSD used)
- ssd-pds-optane-wal (SSD used to store persistence and Optane to store WAL)

The aims of this challenge are to:
- check if it's possible to speed up Apache Ignite using Optane instead of SSD (without Ignite's code modifications)
- prove special PMDK plugin for Ignite provides perfomance boost.

We used P4510 and a couple of Optane 200 series at benchmarks.
Benchmarking configuration was: 1 client with 128 producers and 3 server nodes.

First of all, we benchmarked the in-memory case, and gained the following:
![image](https://user-images.githubusercontent.com/1394154/129332244-dc54138e-5965-4551-a0df-f9398c13621a.png)
This chart represents the maximum possible performance at the deployment.

The second measurement was using SSD with LOG_ONLY WAL mode:
![image](https://user-images.githubusercontent.com/1394154/129332651-32ec074d-a188-4a85-838c-92fcf94f9af2.png)
And results we gain were pretty close to the in-memory case.
