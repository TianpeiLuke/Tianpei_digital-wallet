# PayMo digital wallet solutions
   written by Tianpei (Luke) Xie

## Data Clean
The given batch data are not well-formated, especially the `message` section, which contains extra commas in sentences. Some sentences even take multiple lines which are very hard to read if we load the `batch_payment.csv` file directly into the dataframe. 

We proprocess the data by dropping duplicate rows that only contain sentences from previous `message` section. For extra commas in `message` section, I just maintain a portion of the sentence until its first comma.

In the `./src` directory, I include the `preprocessing()` method in the `read_map.py` file. This method takes the source and filename of the input file and output the `batch_payment_new.csv` in `./paymo_input/` directory. 


## Solution strategy
The key for solving the problem is to find efficient data structures to infer the graph from transaction data.  



### Data structure
Given the transaction data, which include the `id1` as sender, and `id2` as the receiver, we build the adjacency matrix using list of dictionaries, called `adjacency_mat`. Each dictionary has format 

	`{'key': id1, 'neighbor': list of neighbors }`

where the list of neighbors contains all neighbors of `id1` in a transaction network. Similarly for `id2`. Note that each transcation yields two elements in `adjancency_mat`, since the graph is undirected. 

Also, we use a hash table to store the location of each node id in `adjacency_mat`, called `adjacency_index`. `adjacency_index` is a dictionary with keys being the node id, and values being the index of corresponding dictionary element in `adjacency_mat`.

	`adjacency_index = {id1: loc1, id2: loc2, id3: loc3, ... }`

Using `adjacency_index`, we can search the `adjacency_mat` in $O(1)$ time.


### Feature 1: Search for existing edges in graph
To complete the _Feature 1_ requirement, we only need to search if the edge for a new transaction exist in the given graph. 

Using command

	`return (id2 in adjacency_mat[adjacency_index[id1]]['neighbor']) or (id1 in adjacency_mat[adjacency_index[id2]]['neighbor'])`

Note that since the data in `stream_payment.csv` come in sequential order, we process them line by line and adding existing edges to the graph.

### Feature 2: Search for 2nd Order Friend in graph
_Feature 2_ can be accompolished by searching for both the existing edges and the nodes that are connected to the neighbors of the new transaction. 

Using command:

	`any( id2 in adjacency_mat[adjacency_index[x]]['neighbor'] for x in adjacency_mat[adjacency_index[id1]]['neighbor'] )`

