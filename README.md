Just copy repository content to /apache-ignite-X.Y.Z-bin/benchmarks/config/ folder and start one of the following benchmarks:
- inmem (Inmemory mode, no discs used)
- optane1-pds-optane1-wal (Single Optane used)
- optane1-pds-optane2-wal (One Optane used to store persistence and another to store WAL)
- ssd1-pds-ssd1-wal (Single SSD used)
- ssd-pds-optane-wal (SSD used to store persistence and Optane to store WAL)