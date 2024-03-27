---

title: "Dictionary Data Type in MATLAB"
description: ""
tags: [
    "MATLAB",
    "Program",
    "Python",
    "Contribute",
]
date: "2022-09-15"
categories: [
    "docs",
    "Development",
]
author: "AriesChen"
menu:
  main:
    parent: "docs"
    weight: 3

---

---

> A *dictionary* is useful for a fast lookup of values in a larger set. A dictionary is a map that stores data as *values*, which can be accessed using corresponding unique *keys*. Each pair of keys and values is an *entry*.

Before MATLAB R2022b, there is not an official dictionary data type. Though you can use Map Containers as an alternative. It is not very convenient in our daily work. In the version R2022b, the official version is coming. Let’s have a look.

#### Functions

There are some functions in the dictionary object, like entries, keys, values, types, numEntries, isConfigured, and isKey. The content of each function is as felow:

| Functions    | Content                                                      |
| ------------ | ------------------------------------------------------------ |
| entries      | Key-value pairs of dictionary                                |
| keys         | Keys of dictionary                                           |
| values       | Values of dictionary                                         |
| types        | Types of dictionary keys and values                          |
| numEntries   | Number of key-value pairs in dictionary                      |
| isConfigured | Determine if dictionary has types assigned to keys and values |
| isKey        | Determine if dictionary contains key                         |

#### Syntax

The syntax is easy just like in Python and other languages.

```matlab
d = dictionary(keys,values)
d = dictionary(k1,v1,...,kN,vN)
d = dictionary
```

Let's take an example. `d = dictionary(keys, values)` creates a dictionary with specified keys and values. The resulting dictionary `d` is a 1-by-1 scalar object. If multiple values are assigned to the same key, then only the last of those values are assigned. New assignments to an existing key overwrite the value for that entry.

#### Example

I use the dictionary data type to store the serial number and the group label of a seismograph. This helps me to sort and find the information more conveniently.

```matlab
instrGroupSerial = dictionary('590000037','A1', '590000050','A2', '590000059','A3', '590000064','A4',...
                              '590000066','A5', '590000099','A6',...                     
                              '590000102','B1', '590000105','B2', '590000106','B3', '590000111','B4',...
                              '590000342','B5', '590000343','B6',...                     
                              '590000345','C1', '590000351','C2', '590000358','C3', '590000372','C4',...
                              '590000397','C5', '590000400','C6',...                     
                              '590000425','D1', '590000446','D2', '590000450','D3', '590000458','D4',...
                              '590000472','D5', '590000587','D6',...                     
                              '590000075','E1', '590000080','E2', '590000360','E3', '590000894','E4',...
                              '590000904','E5', '590000964','E6',...                     
                              '590001101','F1', '590001170','F2', '590001172','F3', '590001184','F4',...
                              '590001186','F5', '590001231','F6',...                     
                              '590001240','G1', '590001243','G2', '590001300','G3', '590001347','G4',...
                              '590001366','G5', '590001379','G6');
```

If you are interested in the dictionary data type, have a try!
