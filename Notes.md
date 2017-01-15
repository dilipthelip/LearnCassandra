## Three nodes cluster - Replication factor as 3 while creating KeySpace:

In this case the **Owns** Column possess the ownership as 100%.

```
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  208.42 KiB  256          100.0%            f22180f1-76b0-4683-a2f7-4fc2139c285d  rack1
UN  172.17.0.2  274.6 KiB  256          100.0%            a1983ce1-d573-4778-b321-dd9d5bdc56f7  rack1
UN  172.17.0.4  193.24 KiB  256          100.0%            4c1a7281-a214-4a6f-8177-81dc12398308  rack1

```


## Three nodes cluster - Replication factor as 1 while creating KeySpace:

In this case the **Owns** Column split the owneship

```
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  208.42 KiB  256          32.0%             f22180f1-76b0-4683-a2f7-4fc2139c285d  rack1
UN  172.17.0.2  183.89 KiB  256          35.5%             a1983ce1-d573-4778-b321-dd9d5bdc56f7  rack1
UN  172.17.0.4  89.2 KiB   256          32.5%             4c1a7281-a214-4a6f-8177-81dc12398308  rack1

```
