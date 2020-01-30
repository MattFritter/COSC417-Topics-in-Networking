# COSC 416: Topics in Networking
# Winter 2020 - Lab 2.1

In this lab, we'll take the data we generated in Lab 2 and graph it, to make it easier to visualize how the autonomous systems are linked together.

## Table of Contents
- [Handling the Data File](#data)
- [Building our Graph](#graph)
- [Questions](#questions)
- [Submission](#sub)

<a name="data"></a>
## Handling the Data File

We will use the ```asnoutput.txt``` file that we generated in the previous lab. However, you may need to perform some tweaks, as formatting will cause issues when we try to parse the data. You may choose to use Notepad++ or similar text editing software capable of advanced text editing.

Your ```asnoutput.txt``` file should be formatting something similar to this:

```
['154.11.12.219', '852', 'ASN852, CA']
['74.125.50.110', '15169', 'GOOGLE, US']
['74.125.243.177', '15169', 'GOOGLE, US']
['209.85.254.247', '15169', 'GOOGLE, US']
['8.8.8.8', '15169', 'GOOGLE, US']

['154.11.12.219', '852', 'ASN852, CA']
['74.125.50.110', '15169', 'GOOGLE, US']
['74.125.243.177', '15169', 'GOOGLE, US']
['209.85.254.247', '15169', 'GOOGLE, US']
['8.8.4.4', '15169', 'GOOGLE, US']

['154.11.10.11', '852', 'ASN852, CA']
['154.11.15.10', '852', 'ASN852, CA']
['184.105.64.110', '6939', 'HURRICANE, US']
['184.105.148.150', '6939', 'HURRICANE, US']
['207.23.253.117', '271', 'BCNET-AS, CA']
['134.87.30.153', '271', 'BCNET-AS, CA']
['137.82.88.122', '393249', 'UBC, CA']
['137.82.123.113', '393249', 'UBC, CA']
```

Where each line contains a list of IP address, ASN, and AS Description. There should be a single, empty line between each traceroute output, so you should end up with a total of 20 "chunks" of data within your ```asnoutput.txt``` file. If you decided to include errors lines in cases where a router in your traceroute didn't respond, you can leave them in, but it may require some editing of your code to make the graphs work properly.

Once your output is formatted as above, you're ready to begin parsing your data and generating your graph.

<a name="graph"></a>
## Building our Graph (5 marks)

For our graph, we'll be using a Python module called ```networkx```. Networkx, as the name suggests, is designed to produce network graphs. It may already be installed on your system (it comes with many scientific distributions of Python), but if not you can install it using ```pip install networkx```. You will also need to install the Matplotlib library, if you don't already have it: ```pip install matplotlib```.

To begin, we'll import the module and then create our base graph object, *G*:

```
import networkx
G = networkx.Graph()
```

After that, we'll need to open our data file ```asnoutput.txt``` in read mode, and then fetch all the data as a single (very large) string:

```
outputFile = open("asnoutput.txt", "r")
rawText = outputFile.read()
```

We can then perform a ```split()``` operation. In Python, ```split()``` is a string function that will split the list on each instance of the provided string or character, and return a list of the results. Since we are using empty lines to separate each individual trace, we can split on ```/n/n```, which catches the newline from the previous line, plus an empty line containing nothing but the second newline:

```
traces = rawText.split("\n\n")
```

The ```traces``` variable now contains a list, with each item in the list being a single block of entries for one trace, like this:

```
#traces[0]

['154.11.12.219', '852', 'ASN852, CA']
['74.125.50.110', '15169', 'GOOGLE, US']
['74.125.243.177', '15169', 'GOOGLE, US']
['209.85.254.247', '15169', 'GOOGLE, US']
['8.8.8.8', '15169', 'GOOGLE, US']

#traces[1]

['154.11.12.219', '852', 'ASN852, CA']
['74.125.50.110', '15169', 'GOOGLE, US']
['74.125.243.177', '15169', 'GOOGLE, US']
['209.85.254.247', '15169', 'GOOGLE, US']
['8.8.4.4', '15169', 'GOOGLE, US']

...
```

Now, we'll begin iterating over this. We'll need to perform a ```strip()```, which will remove any extra whitespace before and after each block, and then we'll ```split()``` again on a single newline to get individual IP-ASN entries from our trace:

```
for trace in traces:
    traceClean = trace.strip()
	entries = traceClean.split("\n")
```

Within a trace block, we can assume a connection between each entry that is directly adjacent. This means that each entry will have an edge in the graph to the entry that came before it (if there was one) and after it (if there is one). If you have empty entries for non-responsive routers, you'll need to modify the code provided here to account for that.

We'll use a variable called ```previous``` to hold the value of the previously-added node. When we add a new node, we add it to the graph and then drawn an edge between it and the node stored in ```previous```. We can take advantage of the fact that Networkx allows nodes to be identified by a singular value (in our case, the ASN). As a result, we don't have to worry about accidentally creating duplicate nodes (Networkx won't create a duplicate with the same value), and we can easily reference existing nodes by AS in later traces.

We'll set ```previous``` as an empty variable to start, and use the ```eval()``` function to convert our string-representation list into an actual list object:

```
previous = ""
for entry in entries:
   entryList = eval(entry)
```

Note that this goes inside the ```for trace...``` loop. We will set ```previous``` to an empty value at the beginning of each trace block, as at the starting point of our trace there is no previous node. We can then add the current entry to the graph using the ```add_node()``` function. As previously mentioned, this will not create duplicates if the added node already exists in the graph:

```
G.add_node(entryList[1])
```

Note that we are passing it the second value in our entry, which is the ASN number. For this exercise, we don't care about the IP or AS Description too much (although it's useful for adding context).

Finally, we'll check to see if the ```previous``` variable contains a node, and if so we'll create an edge between the current entry, and the ```previous``` entry. We then set ```previous``` to the current node, just before we iterate onto the next entry within the trace:

```
if previous != "":
    G.add_edge(previous, entryList[1])
previous = entryList[1]
```

Then, outside the loop, we can display our graph using the following command:

```
networkx.draw(G, with_labels=True, font_weight='bold')
```

If everything has gone correctly, you should hopefully see something like this outputted when you run your graph (but probably much larger):

<img src="https://i.imgur.com/QBstbX3.png">

<a name="questions"></a>
## Questions (4 marks)

After you've finished your graph, complete the following questions:

1. What is the "starting" ASN for your graph? This will be the ASN that you started each trace from, and should have the most connections.

2. Take a look at one of the better-known ASNs in your graph (like Google, or a major Tier 1 provider). How many connections does it have on your graph? Use an online service to see how many peers the AS actually has - how far off is our graph?

3. Our graph is non-directional, which leaves the assumption that traffic can be sent in both directions between all connected nodes. Given the lecture on routing policy, do you think this is necessarily true?

4. Look at your graph. Based on what we can learn from the graph alone, what would you say is the best predictor for whether a given ASN is a higher-tier service provider?

<a name="sub"></a>
## Submission

When you're done, submit an image copy of your output graph, as well as your four question answers (they can be in the same file), in addition to your Python file via Moodle.
