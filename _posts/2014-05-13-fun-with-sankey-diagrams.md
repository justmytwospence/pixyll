---
author: Spencer Boucher
date: 2014-03-13
layout: post
summary: >
  Let's have some fun with some unconventional data munging and an unconventional
  data visualization technique, using our three favorite tools: Python, R, and D3
title: Fun with Sankey diagrams
---

I'm currently working on the final project for my data visualization course. The
dataset that I've chosen to work with can be [downloaded here][] -- it's a
compendium Reddit resubmissions over a period of several years (ie, images that
were submitted to more than one and/or to multiple subreddits). I waffled for a
long time trying to decide what the best way to visualize the *flow* of images
through various subreddits would be, but just in the nick of time, I stumbled
across Christopher Gandrud's new [d3Network package for R][], and that was
enough cause for me to settle on a Sankey diagram. If you've never heard of
Reddit, the illustrious CPG Grey will enlighten you.

```python
from IPython.display import YouTubeVideo
YouTubeVideo('tlI022aUWQQ')
```

<iframe width="400" height=300" src="https://www.youtube.com/embed/tlI022aUWQQ"
frameborder="0" allowfullscreen ></iframe>

The task of massaging columnar data consisting of an image ID, subreddit name,
and timestamp for each submission into a more networky format suitable for this
type of visualization was interesting enough that I thought it might be a good
post. If nothing else, Christopher's awesome package deserves some love.

Python is my go-to language for data munging of this calibre, so we will use a
Pandas -> NetworkX -> R -> D3 worflow. Without further ado, lets load the Python
modules we will need and take a look at the data.

```python
# Load modules and data

import pandas as pd                        # For reading/munging the data
import networkx as nx                      # For creating a graph structure
from networkx.readwrite import json_graph  # For exporting a graph structure
from itertools import islice               # For some more interesting munging

%load_ext rmagic
from IPython.display import HTML           # To display results when we're done

!csvclean redditSubmissions.csv
d = pd.read_csv('redditSubmissions_out.csv')
```

Note that I first called `csvclean`, from the
[csvkit suite of command line utilities][]. The bang (!) symbol calls the
command line from IPython. `csvclean` fixes a couple of formatting errors in the
original dataset that interfere with R/Panda's parsing functions (something to
do with quotes or commas in the "title" field, I believe). The repaired CSV is
saved with a `_out` prepended to the filename. Nothing fancy is required for the
`read_csv` call in our case.

Now for the hard/interesting part. How do we map the flow of each image
submission through the various subreddits?

 - First we sort by image and (crucially) timetamp on line 4.
 - On line 5, I simply extract the 3 columns that we care about.
 - Now we drop resubmissions of each image to the *same* subreddit with
   drop_duplicates on line 6, which only keeps each image's *first* submission
   to a particular subreddit (why we sorted first).
 - The last thing we need Pandas for is to group by image ID (line 7).

On line 7, we pull the list of subreddits (now unique and nicely ordered) for
each image. The nested list comprehension is necessary only because calling
`.subreddit` on the groupby object `g` returns a tuple by default, and we'd
rather have a list of lists.

```python
# Identify the order in which each image is submitted to various subreddits,
# removing repeats within a subreddit

g = d.sort(['#image_id', 'unixtime'])\
     .ix[:,['#image_id', 'unixtime', 'subreddit']]\
     .drop_duplicates(cols = ['#image_id', 'subreddit'])\
     .groupby('#image_id')
flow = [[el for el in x[1]] for x in list(g.subreddit)]
```

Now we need a function `window` that rolls along each of the lists in `flow` and
connects every subsequent pair of subreddits that a particular image was
submitted to. We'll do this with the help of the wonderful `itertools` module,
creating two dimensional tuples that encode the "from" subreddit and the "to"
subreddit, respectively. In lines 14 and 15, we apply the function and flatten
the result to a single list.

In order to truly capture the "flow," however, we need to distinguish between
the "gifs" subreddit node where images are popping up for the first time and the
"gifs" subreddit node when the image has already appeared in another subreddit
(say, "pics"). The `enumerate` in line 14 does this by tacking on the ordinality
to the name of the node, admittedly very hacky, but we have a lot of tuples
floating around already.

```python
# Roll along the list of subreddits each image has been submitted to,
# creating an edge tuple for each subsequent pair
def window(seq, n=2):
    '''Returns a sliding window (of width n) over data from the iterable
       s -> (s0,s1,...s[n-1]), (s1,s2,...,sn), ...'''
    it = iter(seq)
    result = tuple(islice(it, n))
    if len(result) == n:
        yield result
    for elem in it:
        result = result[1:] + (elem,)
        yield result

sankey = [list(window([str(i) + x for i, x in enumerate(sub)])) for sub in flow]
sankey = [item for sublist in sankey for item in sublist] # flatten
```

At last we have a list of edges that on some level describes the flow that we
are trying to get at. Now we can just iterate through them and use `NetworkX` to
create the graph and weight the edges appropriately. In lines 10--13, I prune
back the tiny edges that clutter up the diagram, and then the nodes that are no
longer associated with any edges. Last but not least, we export the structure to
a JSON in line 16.

```python
# Create network structure
S = nx.DiGraph()
for edge in sankey:
    if S.has_edge(*edge):
        S[edge[0]][edge[1]]['weight'] +=1
    else:
        S.add_edge(*edge, weight = 1)

# Trim edges
S.remove_edges_from([x for x in S.edges(data=True) if x[2]['weight'] < 50])
flagged = [x for x, el in S.out_degree().items() if (x[0] != '3') & (el == 0)]
S.remove_edges_from([x for x in S.edges(data=True) if x[1] in flagged])
S.remove_nodes_from([x for x, n in S.degree().items() if n == 0])

# Export
json_graph.dump(S, open('sankey.json', 'w'))
```

Time for R!

We will need to make sure that `d3Network` is installed to the the instance of R
that is used by IPython's `Rmagic` via `Rpy2`. It was different for me (I
think?) so if you are running something like this for the first time, include
the lines that are commented out.

The `%%R` denotes block-level R magicks in IPython (`%R` will give you
line-level magicks)

```R
%%R
install.packages('devtools')
library(devtools)
devtools::install_github("christophergandrud/d3Network")
library(d3Network)
```

This is finally the point at which Christopher Gandrud's package simplifies
everything for us. We simply read in the nodes and linkes (edges) from the JSON
file (they get converted to two dataframes). Note that we have to strip the
janky ordinality numbers that we tacked onto the node names (line 3). Now that
different nodes have the same names, the package will even make sure that each
subreddit node has the same color every time it appears!

The call to `d3Sankey` points to the the nodes dataframe, the links dataframe,
the name of the sources/targets in the links dataframe, the name of the column
that holds the link weights, and then some display configuration stuff.

```R
%%R
nodes <- JSONtoDF(file = paste0('sankey.json'), array = 'nodes')
nodes$id <- substring(nodes$id, 2)
links <- JSONtoDF(file = paste0('sankey.json'), array = 'links')
d3Sankey(Nodes = nodes, Links = links, Source = 'source',
         Target = 'target', Value = 'weight', NodeID = 'id',
         width = 600, height = 500, fontsize = 12,
         standAlone = TRUE, iframe = TRUE, file = 'sankey.html')
```

We can render the resulting iframe directly in our IPython notebook! Hover over
edges for some nice brushing or click and drag the nodes to untangle a
relationship you're interested in.

```python
HTML('<iframe src="./sankey.html" height=540 width=700 frameBorder="0"></iframe>')
```

<iframe src="{{ site.url }}/images/sankey.html" height=540 width=700 frameBorder="0"></iframe>

This type of "tiered" Sankey diagram is a little unconventional, but so far its
the best way I can come up with to visualize the interesting phenomenon of
submission flow through Reddit. Leave a comment if this gives you any
interesting ideas, I'd love to hear them!

[downloaded here]: http://snap.stanford.edu/data/web-Reddit.html

[d3Network package for R]: http://christophergandrud.github.io/d3Network/

[csvkit suite of command line utilities]: http://csvkit.readthedocs.org/en/latest/index.html
