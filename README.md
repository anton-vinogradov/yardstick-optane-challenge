# HOWTO
Just copy repository content to apache-ignite-X.Y.Z-bin/benchmarks/config/ folder and start one of the following benchmarks:
- inmem (Inmemory mode, no discs used)
- optane1-pds-optane1-wal (Single Optane used)
- optane1-pds-optane2-wal (One Optane used to store persistence and another to store WAL)
- ssd1-pds-ssd1-wal (Single SSD used)
- ssd-pds-optane-wal (SSD used to store persistence and Optane to store WAL)

# AIM
The aims of this challenge are to:
- check if it's possible to speed up Apache Ignite using Optane instead of SSD (without Ignite's code modifications)
- demonstrate that a special PMDK plugin for Ignite provides a performance boost

We used P4510 and a couple of Optane 200 series at benchmarks. 
Benchmarking configuration was: 1 client with 128 producers and 3 server nodes.

First of all, we benchmarked the in-memory case, and gained the following:
![image](https://user-images.githubusercontent.com/1394154/129332244-dc54138e-5965-4551-a0df-f9398c13621a.png)
This chart represents the maximum possible performance at the deployment.

The second measurement was using SSD with LOG_ONLY WAL mode:
![image](https://user-images.githubusercontent.com/1394154/129332651-32ec074d-a188-4a85-838c-92fcf94f9af2.png)
And results we gain were pretty close to the in-memory case.

After this we benchmarked SSD with FSYNC WAL mode:
![image](https://user-images.githubusercontent.com/1394154/129333006-2b92cfbe-3761-484a-81a5-5c3bbe1d42cb.png)
And gain huge performance drop at puts (149k -> 7k).

And it's time to check if Optane is faster than SSD when used as SSD (FSYNC WAL mode):
![image](https://user-images.githubusercontent.com/1394154/129333298-ffd86b97-70a8-49cd-9a76-066c11b1df0f.png)
And we found it provides the same performance as a good SSD.

The last benchmark we did was checking Optain's couple, first was responsible for persistence, and second for WAL (FSYNC WAL mode):
![image](https://user-images.githubusercontent.com/1394154/129333609-260b391d-0142-48ab-8e2c-89c5caab191a.png)
And we got the same numbers.

What it finally means:
- Optane as fast as good SSD (Optane usage as SSD gives you no additional boost).
- Optane should be used via PMDK to gain a real boost (we did some PMDK checks and found a boost is possible).

# PMDK check
To check the performance boost gained by Ignite PMDK plugin we're going to develop, we created a special benchmark 'puts', which compares puts speed at the in-memory, SSD, and Optane (both with FSYNC WAL mode).
Current results (without PMDK plugin):

# Helpful scripts
As written before, we used 1 client node and 3 servers.
Lest define client node located at localhost and server nodes at 10.0.0.2, 10.0.0.3, 10.0.0.4
To create all necessary folders use:
```
ssh 10.0.0.2 'sudo mkdir /mnt/pmem0/persistence; sudo chmod 777 /mnt/pmem0/persistence; sudo mkdir /mnt/pmem0/wal; sudo chmod 777 /mnt/pmem0/wal; sudo mkdir /mnt/pmem1/wal; sudo chmod 777 /mnt/pmem1/wal; sudo mkdir /mnt/ssd/persistence; sudo chmod 777 /mnt/ssd/persistence; sudo mkdir /mnt/ssd/wal; sudo chmod 777 /mnt/ssd/wal;'; ssh 10.0.0.3 'sudo mkdir /mnt/pmem0/persistence; sudo chmod 777 /mnt/pmem0/persistence; sudo mkdir /mnt/pmem0/wal; sudo chmod 777 /mnt/pmem0/wal; sudo mkdir /mnt/pmem1/wal; sudo chmod 777 /mnt/pmem1/wal; sudo mkdir /mnt/ssd/persistence; sudo chmod 777 /mnt/ssd/persistence; sudo mkdir /mnt/ssd/wal; sudo chmod 777 /mnt/ssd/wal;'; ssh 10.0.0.4 'sudo mkdir /mnt/pmem0/persistence; sudo chmod 777 /mnt/pmem0/persistence; sudo mkdir /mnt/pmem0/wal; sudo chmod 777 /mnt/pmem0/wal; sudo mkdir /mnt/pmem1/wal; sudo chmod 777 /mnt/pmem1/wal; sudo mkdir /mnt/ssd/persistence; sudo chmod 777 /mnt/ssd/persistence; sudo mkdir /mnt/ssd/wal; sudo chmod 777 /mnt/ssd/wal;'
```
The folder should be cleaned between checks, to do so use:
```
ssh 10.0.0.2 'rm -r apache-ignite-2.10.0-bin; rm -r /mnt/pmem0/persistence/*; rm -r /mnt/pmem1/wal/*; rm -r /mnt/pmem0/wal/*; rm -r /mnt/ssd/persistence/*; rm -r /mnt/ssd/wal/*; ps aux | grep java' ; ssh 10.0.0.3 'rm -r apache-ignite-2.10.0-bin; rm -r /mnt/pmem0/persistence/*; rm -r /mnt/pmem1/wal/*; rm -r /mnt/pmem0/wal/*; rm -r /mnt/ssd/persistence/*; rm -r /mnt/ssd/wal/*; ps aux | grep java' ; ssh 10.0.0.4 'rm -r apache-ignite-2.10.0-bin; rm -r /mnt/pmem0/persistence/*; rm -r /mnt/pmem1/wal/*; rm -r /mnt/pmem0/wal/*; rm -r /mnt/ssd/persistence/*; rm -r /mnt/ssd/wal/*; ps aux | grep java'
```
