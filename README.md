[![license](https://img.shields.io/badge/license-Apache_2.0-blue.svg)](https://github.com/jshalaby510/PySparkIP/blob/main/LICENSE)

# PySparkIP
An API for working with IP addresses in Apache Spark.

## Usage
  * pip install PySparkIP
  * from SparkIP import *

## License
This project is licensed under the Apache License. Please see [LICENSE](LICENSE) file for more details.

## Tutorial
### Initialize
Before using in SparkSQL, initialize PySparkIP by passing `spark` to `SparkIPInit`.
```python
from pyspark.sql import SparkSession
from SparkIP import *

spark = SparkSession.builder.appName("ipTest").getOrCreate()
SparkIPInit(spark)
```

### Functions
**Check address type**
```python
# Multicast
spark.sql("SELECT * FROM IPAddresses WHERE isMulticast(IPAddress)")
ipDF.select('*').filter("isMulticast(IPAddress)")
ipDF.select('*').withColumn("IPColumn", isMulticast("IPAddress"))

"""
Other address types:
    isPrivate, isGlobal, isUnspecified, isReserved, 
    isLoopback, isLinkLocal, isIPv4Mapped, is6to4, 
    isTeredo, isIPv4, isIPv6
"""
```

**Output address in different formats**
```python
# Exploded
spark.sql("SELECT explodedIP(IPAddress) FROM IPAddresses")
ipDF.select(explodedIP("IPAddress"))

# Compressed
spark.sql("SELECT compressedIP(IPAddress) FROM IPAddresses")
ipDF.select(compressedIP("IPAddress"))
```

**Sort IP Addresses**
```python
# SparkSQL doesn't support values > LONG_MAX
# To sort IPv6 addresses, use ipAsBinary
# To sort IPv4 addresses, use either ipv4AsNum or ipAsBinary, but ipv4AsNum is more efficient

# Sort IPv4 and IPv6
spark.sql("SELECT * FROM IPAddresses SORT BY ipAsBinary(IPAddress)")
ipDF.select('*').sort(ipAsBinary("IPAddress"))

# Sort ONLY IPv4
spark.sql("SELECT * FROM IPv4 SORT BY ipv4AsNum(IPAddress)")
ipv4DF.select('*').sort(ipv4AsNum("IPAddress"))
```

**IP network functions**
```python
# Network contains
spark.sql("SELECT * FROM IPAddresses WHERE networkContains(IPAddress, '195.0.0.0/16')")
ipDF.select('*').filter("networkContains(IPAddress, '195.0.0.0/16')")
ipDF.select('*').withColumn("netCol", networkContains("192.0.0.0/16")("IPAddress"))
```

**IP Set**
#### Create IP Sets:
```python
ipStr = '192.0.0.0'
netStr = '225.0.0.0'
ip_net_mix = ('::5', '5.0.0.0/8', '111.8.9.7')

ipSet = IPSet(ipStr, '::/16', '2001::', netStr, ip_net_mix)
ipSet2 = IPSet("6::", "9.0.8.7")
ipSet3 = IPSet(ipSet, ipSet2)
ipSet4 = IPSet()
```
#### Register IP Sets for use in SparkSQL:
Before using IP Sets in SparkSQL, register it by passing it to `SparkIPSets`
```python
ipSet = IPSet('::')
ipSet2 = IPSet()

# Pass the set, then the set name
SparkIPSets.add(ipSet, 'ipSet')
SparkIPSets.add(ipSet2, 'ipSet2')
```
#### Remove IP Sets from registered sets in SparkSQL:
```python
SparkIPSets.remove('ipSet', 'ipSet2')
```

#### Use IP Sets in SparkSQL:
```python
# Note you have to pass the variable name using SparkSQL, not the actual variable

# Initialize an IP Set
setOfIPs = {"192.0.0.0", "5422:6622:1dc6:366a:e728:84d4:257e:655a", "::"}
ipSet = IPSet(setOfIPs)

# Register it
SparkIPSets.add(ipSet, 'ipSet')

#Use it!
# Set Contains
spark.sql("SELECT * FROM IPAddresses WHERE setContains(IPAddress, 'ipSet')")

# Show sets available to use
SparkIPSets.setsAvailable()

# Remove a set
SparkIPSets.remove('ipSet')

# Clear sets available
SparkIPSets.clear()
```

#### IP Set functions (outside of SparkSQL):
```python
ipSet = IPSet()

# Add
ipSet.add('0.0.0.0', '::/16')

# Remove
ipSet.remove('::/16')

# Contains
ipSet.contains('0.0.0.0', '::')

# Clear
ipSet.clear()

# Show all
ipSet.showAll()

# Union
ipSet2 = ('2001::', '::33', 'ffff::f')
ipSet.union(ipSet2)

# Intersects
ipSet.intersects(ipSet2)

# Diff
ipSet.diff(ipSet2)

# Is empty
ipSet.isEmpty()
```
