[![Build Status](https://img.shields.io/travis/abusix/ahocorapy/master.svg)](https://travis-ci.org/abusix/ahocorapy)
[![Test Coverage](https://img.shields.io/coveralls/github/abusix/ahocorapy/master.svg)](https://coveralls.io/github/abusix/ahocorapy)
[![PyPi Version](https://img.shields.io/pypi/v/ahocorapy.svg)](https://pypi.python.org/pypi/ahocorapy)
[![PyPi License](https://img.shields.io/pypi/l/ahocorapy.svg)](https://pypi.python.org/pypi/ahocorapy)
[![PyPi Versions](https://img.shields.io/pypi/pyversions/ahocorapy.svg)](https://pypi.python.org/pypi/ahocorapy)
[![PyPi Wheel](https://img.shields.io/pypi/wheel/ahocorapy.svg)](https://pypi.python.org/pypi/ahocorapy)

# ahocorapy - Fast Many-Keyword Search in Pure Python

ahocorapy is a pure python implementation of the Aho-Corasick Algorithm.
Given a list of keywords one can check if at least one of the keywords exist in a given text in linear time.

## Comparison:

### Why another Aho-Corasick implementation?

We started working on this in the beginning of 2016. Our requirements included unicode support combined with python2.7. That
was impossible with C-extension based libraries (like [pyahocorasick](https://github.com/WojciechMula/pyahocorasick/)). Pure 
python libraries were very slow or unusable due to memory explosion. Since then another pure python library was released 
[py-aho-corasick](https://github.com/JanFan/py-aho-corasick). The repository also contains some discussion about different
implementations. 
There is also [acora](https://github.com/scoder/acora), but it includes the note ('current construction algorithm is not 
suitable for really large sets of keywords') which really was the case the last time I tested, because RAM ran out quickly.

### Differences

- Compared to [pyahocorasick](https://github.com/WojciechMula/pyahocorasick/) our library supports unicode in python 2.7 just like [py-aho-corasick](https://github.com/JanFan/py-aho-corasick).
We don't use any C-Extension so the library is not platform dependant.

- On top of the standard Aho-Corasick longest suffix search, we also perform a shortcutting routine in the end, so
that our lookup is fast while, the setup takes longer. During set up we go through the states and directly add transitions that are
"offered" by the longest suffix or their longest suffixes. This leads to faster lookup times, because in the end we only have to
follow simple transitions and don't have to perform any additional suffix lookup. It also leads to a bigger memory footprint,
because the number of transitions is higher, because they are all included explicitely and not implicitely hidden by suffix pointers.

- We added a small tool that helps you visualize the resulting graph. This may help understanding the algorithm, if you'd like. See below.

### Performance

I compared the two libraries mentioned above with ahocorapy. We used 50,000 keywords long list and an input text of 34,199 characters.
In the text only one keyword of the list is contained.
The setup process was run once per library and the search process was run 100 times. The following results are in seconds (not averaged for the lookup).

You can perform this test yourself using `python tests/ahocorapy_performance_test.py`. (Except for the pyahocorasick_py results. These were taken by importing the
pure python version of the code of [pyahocorasick](https://github.com/WojciechMula/pyahocorasick/). It's not available through pypi
as stated in the code.)

I also added measurements for the pure python libraries with run with pypy.

These are the results:

| Library (Variant)                                  | Setup (1x) | Search (100x) |
|----------------------------------------------------|-:-:--------|--:------------|
| ahocorapy                                          | 1.3s       | 1.51s         |
| ahocorapy (run with pypy)                          | 1.3s       | 0.09s         |
| pyahocorasick                                      | 0.1s       | 0.06s         |
| pyahocorasick (pure python variant in github repo) | 0.5s       | 1.68s         |
| py_aho_corasick                                    | 1.9s       | 13.2s         |
| py_aho_corasick (run with pypy)                    | 1.3s       | 3.71s         |

As expected the C-Extension shatters the pure python implementations. Even though there is probably still room for optimization in
ahocorapy we are not going to get to the mark that pyahocorasick sets. ahocorapy's lookups are faster than py_aho_corasick. 
When run with pypy [PyPy 5.1.2 with GCC 5.3.1 20160413] ahocorapy is almost as fast as pyahocorasick, at least when it comes to
searching. The setup overhead is higher due to the suffix shortcutting mechanism used.


## Basic Usage:

### Creation of the Search Tree

```python
from ahocorapy.keywordtree import KeywordTree
kwtree = KeywordTree(case_insensitive=True)
kwtree.add('malaga')
kwtree.add('lacrosse')
kwtree.add('mallorca')
kwtree.add('mallorca bella')
kwtree.add('orca')
kwtree.finalize()
```

### Searching

```python
result = kwtree.search('My favorite islands are malaga and sylt.')
print(result)
```

Prints :
```python
('malaga', 24)
```

The search_all method returns a generator for all keywords found, or None if there is none.

```python
results = kwtree.search_all('malheur on mallorca bellacrosse')
for result in results:
    print(result)
```

Prints :
```python
('mallorca', 11)
('orca', 15)
('mallorca bella', 11)
('lacrosse', 23)
```

## Drawing Graph

You can print the underlying graph with the Visualizer class.
This feature requires a working pygraphviz library installed.

```python
from ahocorapy_visualizer.visualizer import Visualizer
visualizer = Visualizer()
visualizer.draw('readme_example.png', kwtree)
```

The resulting .png of the graph looks like this: 

![graph for kwtree](https://raw.githubusercontent.com/abusix/ahocorapy/master/img/readme_example.png "Keyword Tree")

