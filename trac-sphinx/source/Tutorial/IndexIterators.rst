.. sidebar:: ToC

   .. contents::


.. _tutorial-index-iterators:

Index Iterators
---------------

Learning Objective
  You will know the different kinds of index indices and how to use them for searching.

Difficulty
  Average

Duration
  1.5 h

Prerequisites ::
  :ref:`tutorial-sequences`, :ref:`tutorial-iterators`

Virtual String Tree Iterator
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The search procedure of :dox:`IndexEsa`, :dox:`IndexWotd IndexWotd`, :dox:`IndexDfi` and :dox:`FMIndex` are suffix array based.
This can be utilized in form of a common iterator interface.
This common interface is the Virtual String Tree Iterator (:dox:`VSTreeIterator VSTree Iterator`) in SeqAn, which lets you access the :dox:`IndexEsa`, :dox:`IndexWotd` and :dox:`IndexDfi IndexDfi` as if using a suffix tree (`suffix tree definition <Tutorial/Indices/SuffixTree>`__) and the :dox:`FMIndex FMIndex` as if using a prefix trie.

In the first part of this tutorial we will concentrate on the :dox:`TopDownIterator TopDown Iterator` which is one of the two index iterator specializations (besides the :dox:`BottomUpIterator BottomUp Iterator`).
The second part will then deal with the DFS.

Top-Down Iteration
~~~~~~~~~~~~~~~~~~

For index based pattern search or algorithms traversing only the upper parts of the suffix tree the :dox:`TopDownIterator#TopDown Iterator` or :dox:`TopDownHistoryIterator#TopDown History Iterator` is the best solution.
Both provide the functions :dox:`VSTreeIterator#goDown goDown` and :dox:`VSTreeIterator#goRight goRight` to go down to the first child node or go to the next sibling.
The :dox:`TopDownHistoryIterator#TopDown History Iterator` additionally provides :dox:`TopDownHistoryIterator#goUp goUp` to go back to the parent node.
The child nodes in :dox:`IndexEsa IndexEsa` indices are lexicographically sorted from first to last.
For :dox:`IndexWotd` and :dox:`IndexDfi` indices this holds for all children except the first.

In the next example we want to use the :dox:`TopDownIterator#TopDown Iterator` to efficiently search a text for exact matches of a pattern.
We therefore want to use :dox:`VSTreeIterator#goDown goDown` which has an overload to go down an edge beginning with a specific character.

.. important::

   Note that the iterator traverses the complete edge.
   It does not stop after the first characters if the edge represents more than one character.
   This is true for all tree iterators.
   The only exception is the iterator of the :dox:`FMIndex`, which is a trie iterator.

First we create an index of the text ``"How much wood would a woodchuck chuck?"``

.. includefrags:: core/demos/tutorial/index/index_search.cpp
   :fragment: initialization

Afterwards we create the :dox:`TopDownIterator#TopDown Iterator` using the metafunction Iterator, which expects two arguments, the type of the container to be iterated and a specialization tag (see the VSTree Iterator hierarchy and the :dox:`tutorial-iterators` Tutorial for more details).

.. includefrags:: core/demos/tutorial/index/index_search.cpp
   :fragment: iterator

The main search can then be implemented using the functions :dox:`VSTreeIterator#repLength repLength` and :dox:`VSTreeIterator#representative representative`.
Since :dox:`VSTreeIterator#goDown goDown` might cover more than one character it is necessary to compare parts of the pattern against the representative of the iterator.
The search can now be implemented as follows.
The algorithm descends the suffix tree along edges beginning with the corresponding pattern character.
In each step the ``unseen`` edge characters have to be verified.

.. includefrags:: core/demos/tutorial/index/index_search.cpp
   :fragment: iteration

If all pattern characters could successfully be compared we end in the topmost node who's leaves point to text positions starting with the pattern.
Thus, the suffixes represented by this node are the occurrences of our pattern and can be retrieved with :dox:`VSTreeIterator#getOccurrences getOccurrences`.

.. includefrags:: core/demos/tutorial/index/index_search.cpp
   :fragment: output

Program output:

.. code-block:: console

   w
   wo
   wood
   9
   22

Alternatively, we could have used :dox:`VSTreeIterator#goDown goDown` to go down the path of a pattern instead single characters:

.. includefrags:: core/demos/tutorial/index/index_search2.cpp
   :fragment: output

.. code-block:: console

   9
   22

Assignment 1
""""""""""""

.. container:: assignment

   Type
     Review

   Objective
     Copy the code into a demo program and replace the text with a string set containing the strings ``"How much"``, ``"wood would"`` and ``" a woodchuck chuck?"``.

   Solution
     .. container:: foldable

        .. includefrags:: core/demos/tutorial/index/iterator_solution1.cpp

	The difference is the format of the positions of the found occurrences.
        Here, we need a :dox:`Pair` to indicate the string within the :dox:`StringSet` and a position within the string.

Assignment 2
""""""""""""

.. container:: assignment

   Type
     Review

   Objective
     Write a little program that traverses the nodes of the suffix tree of ``"tobeornottobe"`` in the order shown here:

     .. image:: streePreorder.png
	:align: center
	:width: 300px

     At each node print the text of the edges from the root to the node.
     You may only use the functions :dox:`VSTreeIterator#goDown goDown`, :dox:`VSTreeIterator#goRight goRight`, :dox:`TopDownHistoryIterator#goUp goUp` and :dox:`VSTreeIterator#goRoot goRoot`, :dox:`VSTreeIterator#isRoot isRoot` and :dox:`VSTreeIterator#representative representative` which returns the string that represents the node the iterator points to.

   Hint
     * Use a :dox:`TopDownHistoryIterator#TopDown History Iterator`.
     * The code skeleton could look like this:

       .. code-block:: cpp

	  #include <iostream>
	  #include <seqan/index.h>

	  using namespace seqan;

	  int main ()
	  {
	      typedef Index<CharString> TIndex;
	      TIndex index("tobeornottobe");
	      Iterator< TIndex, TopDown<ParentLinks<> > >::Type it(index);

	      do {
		  ...
	      } while (isRoot(it));

	      return 0;
	  }

   Solution
     .. container:: foldable

        .. includefrags:: core/demos/tutorial/index/iterator_solution2.cpp

Assignment 3
""""""""""""

.. container:: assignment

   Type
     Review

   Objective
     Modify the program to efficiently skip nodes with representatives longer than 3.
     Move the whole program into a template function whose argument specifies the index type and call this function twice, once for the :dox:`IndexEsa` and once for the :dox:`IndexWotd` index.

   Solution
     .. container:: foldable 

	We modify the DFS traversal to skip the descent if we walk into a node whose representative is longer than 3.
	We then proceed to the right and up as long as the representative is longer than 3.

	.. includefrags:: core/demos/tutorial/index/index_assignment4.cpp
	   :fragment: iteration

	.. code-block:: console

	   be
	   e
	   o
	   obe
	   t


	   be
	   e
	   o
	   obe
	   t

Depth-First Search
~~~~~~~~~~~~~~~~~~

The tree traversal in assignment 2 is equal to a the tree traversal in a full depth-first search (dfs) over all suffix tree nodes beginning either in the root (preorder dfs) or in a leaf node (postorder dfs).
A preorder traversal (:ref:`figure-stree-preorder`) halts in a node when visiting it for the first time whereas a postorder traversal (:ref:`figure-stree-postorder`) halts when visiting a node for the last time.
The following two figures give an example in which order the tree nodes are visited.

.. _figure-stree-preorder:

.. figure:: streePreorder.png
   :align: left

   Preorder DFS

.. _figure-stree-postorder:

.. figure:: streePreorder.png
   :align: right

   Postorder DFS

Since these traversals are frequently needed SeqAn provides special iterators which will we describe next.

We want to construct the suffix tree of the string "abracadabra" and output the substrings represented by tree nodes in preorder dfs.
In order to do so, we create the string "abracadabra" and an index specialized with the type of this string.

.. includefrags:: core/demos/tutorial/index/index_preorder.cpp
   :fragment: includes

The :dox:`Iterator` metafunction expects two arguments, the type of the container to be iterated and a specialization tag, as described earlier.
In this example we chose a :dox:`TopDownHistoryIterator#TopDown History Iterator` whose signature in the second template argument is ``TopDown< ParentLinks<Preorder> >``.

.. includefrags:: core/demos/tutorial/index/index_preorder.cpp
   :fragment: initialization

As all DFS suffix tree iterators implement the :dox:`VSTreeIterator VSTree Iterator`, they can be used via :dox:`VSTreeIterator#goNext goNext`, :dox:`VSTreeIterator#atEnd atEnd`, etc.

.. includefrags:: core/demos/tutorial/index/index_preorder.cpp
   :fragment: iteration

Program output:

.. code-block:: console

   a
   abra
   abracadabra
   acadabra
   adabra
   bra
   bracadabra
   cadabra
   dabra
   ra
   racadabra

.. tip::

   There are currently 2 iterators in SeqAn supporting a DFS search:

   +-------------------------+----------+-----------+
   | Iterator                | Preorder | Postorder |
   +=========================+==========+===========+
   | :dox:`BottomUpIterator` | no       | yes       |
   +-------------------------+----------+-----------+
   | :dox:`HistoryIterator`  | yes      | yes       |
   +-------------------------+----------+-----------+


   If solely a postorder traversal is needed the :dox:`BottomUpIterator BottomUp Iterator` should be preferred as it is more memory efficient.
   Please note that the BottomUp Iterator is only applicable to :dox:`IndexEsa` indices.

.. TODO: Dave has to integrate teh SuffixTree definition here.

.. tip:: 

   A relaxed suffix tree (see [[Tutorial/Indices/SuffixTree| definition]]) is a suffix tree after removing the $ characters and empty edges.
   For some bottom-up algorithms it would be better not to remove empty edges and to have a one-to-one relationship between leaves and suffices.
   In that cases you can use the tags PreorderEmptyEdges or PostorderEmptyEdges instead of Preorder or Postorder or EmptyEdges for the TopDown Iterator.

Note that the :dox:`VSTreeIterator#goNext goNext` is very handy as it simplifies the tree traversal in assignment 2 greatly.

Assignment 4
""""""""""""

.. container:: assignment

   Type
     Review

   Objective
     Write a program that constructs an index of the :dox:`VSTreeIterator#StringSet StringSet` "tobeornottobe", "thebeeonthecomb", "beingjohnmalkovich" and outputs the strings corresponding to suffix tree nodes in postorder DFS.

   Solution
     .. container:: foldable

	First we have to create a :dox:`Class.StringSet` of :dox:`Shortcut.CharString` (shortcut for ``String<char>``) and append the 3 strings to it.
	This could also be done by using :dox:`Function.resize` and :dox:`Function.assignValue`.
	The first template argument of the index class has to be adapted and is now a StringSet.

	.. includefrags:: core/demos/tutorial/index/index_assignment1.cpp
	   :fragment: initialization

	To switch to postorder DFS we have two change the specialization tag of ``ParentLinks`` from ``Preorder`` to ``Postorder``.
	Please note that the :dox:`Spec.TopDown Iterator` always starts in the root node, which is the last postorder DFS node.
	Therefore, the iterator has to be set explicitly to the first DFS node via :dox:`Function.goBegin`.

	.. includefrags:: core/demos/tutorial/index/index_assignment1.cpp
	   :fragment: iteration1

	Alternatively to a :dox:`TopDownHistoryIterator` you also could have used a :dox:`BottomUpIterator` with the same result.
	The BottomUp Iterator automatically starts in the first DFS node as it supports no random access.

	.. includefrags:: core/demos/tutorial/index/index_assignment1.cpp
	   :fragment: iteration2

	Program output:

	.. code-block:: console

	   alkovich
	   beeonthecomb
	   beingjohnmalkovich
	   beornottobe
	   be
	   b
	   ch
	   comb
	   c
	   ebeeonthecomb
	   ecomb
	   eeonthecomb
	   eingjohnmalkovich
	   eonthecomb
	   eornottobe
	   eo
	   e
	   gjohnmalkovich
	   hebeeonthecomb
	   hecomb
	   he
	   hnmalkovich
	   h
	   ich
	   ingjohnmalkovich
	   i
	   johnmalkovich
	   kovich
	   lkovich
	   malkovich
	   mb
	   m
	   ngjohnmalkovich
	   nmalkovich
	   nottobe
	   nthecomb
	   n
	   obeornottobe
	   obe
	   ohnmalkovich
	   omb
	   onthecomb
	   ornottobe
	   ottobe
	   ovich
	   o
	   rnottobe
	   thebeeonthecomb
	   thecomb
	   the
	   tobeornottobe
	   tobe
	   ttobe
	   t
	   vich

As a last assignment lets try out one of the specialised iterators, which you can find at the bottom of this page.
Look there for the specialisation which iterates over all maximal unique matches (MUMS).

Assignment 5
""""""""""""

.. container:: assignment

   Type
     Review

   Objective
     Write a program that outputs all maximal unique matches (MUMs) between ``"CDFGHC"`` and ``"CDEFGAHC"``.

   Solution
     .. container:: foldable

	Again, we start to create a :dox:`StringSet` of :dox:`CharString` and append the 2 strings.

	.. includefrags:: core/demos/tutorial/index/index_assignment2.cpp
	   :fragment: initialization

	After that we simply use the predefined iterator for searching MUMs, the :dox:`MumsIterator`.
	Its constructor expects the index and optionally a minimum MUM length as a second parameter.
	The set of all MUMs can be represented by a subset of suffix tree nodes.
	The iterator will halt in every node that is a MUM of the minimum length.
	The corresponding match is the node's seqan:Function.representative.

	.. includefrags:: core/demos/tutorial/index/index_assignment2.cpp
	   :fragment: iteration

	Program output:

	.. code-block:: console

	   CD
	   FG
	   HC

Accessing Suffix Tree Nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the previous subsection we have seen how to walk through a suffix tree.
We now want to know what can be done with a suffix tree iterator.
As all iterators are specializations of the general VSTree Iterator class, they inherit all of its functions.
There are various functions to access the node the iterator points at (some we have already seen), so we concentrate on the most important ones.

:dox:`VSTreeIterator#representative representative`
  returns the substring that represents the current node, i.e. the concatenation of substrings on the path from the root to the current node

:dox:`VSTreeIterator#getOccurrence getOccurrence`
  returns a position where the representative occurs in the text

:dox:`VSTreeIterator#getOccurrences getOccurrences`
  returns a string of all positions where the representative occurs in the text

.. TODO: Dave has to integrate the figure here.

:dox:`VSTreeIterator#isRightTerminal isRightTerminal`
  tests if the representative is a suffix in the text (corresponds to the shaded nodes in the suffix tree figures)

:dox:`VSTreeIterator#isLeaf isLeaf`
  tests if the current node is a tree leaf

:dox:`VSTreeIterator#parentEdgeLabel parentEdgeLabel`
  returns the substring that represents the edge from the current node to its parent (only TopDownHistory Iterator)

.. TODO: Dave has to integrate the figure here

.. important::

   There is a difference between the functions isLeaf and isRightTerminal.
   In a relaxed suffix tree (see `definition <Tutorial/Indices/SuffixTree>`__) a leaf is always a suffix, but not vice versa, as there can be internal nodes a suffix ends in.
   For them isLeaf returns false and isRightTerminal returns true.

Property Maps
~~~~~~~~~~~~~

Some algorithms require to store auxiliary information (e.g. weights, scores) to the nodes of a suffix tree.
To attain this goal SeqAn provides so-called property maps, simple Strings of a property type.
Before storing a property value, these strings must first be resized with :dox:`Index#resizeVertexMap resizeVertexMap`.
The property value can then be assigned or retrieved via seqan:Function.assignProperty or seqan:Function.getProperty, seqan:Function.property.
It is recommended to call :dox:`Index#resizeVertexMap resizeVertexMap` prior to every call of seqan:Function.assignProperty to ensure that the property map has sufficient size.
The following example iterates over all nodes in preorder dfs and recursively assigns the node depth to each node.
First we create a :dox:`String` of ``int`` to store the node depth for each suffix tree node.

.. includefrags:: core/demos/tutorial/index/index_property_maps.cpp
   :fragment: initialization

The main loop iterates over all nodes in preorder DFS, i.e. parents are visited prior children.
The node depth for the root node is 0 and for all other nodes it is the parent node depth increased by 1.
The functions seqan:Function.assignProperty, seqan:Function.getProperty and seqan:Function.property must be called with a seqan:Metafunction.VertexDescriptor.
The vertex descriptor of the iterator node is returned by seqan:Function.value and the descriptor of the parent node is returned by seqan:Function.nodeUp.

.. includefrags:: core/demos/tutorial/index/index_property_maps.cpp
   :fragment: iteration

At the end we again iterate over all nodes and output the calculated node depth.

.. includefrags:: core/demos/tutorial/index/index_property_maps.cpp
   :fragment: output

Program output:

.. code-block:: console

    0
    1       a
    2       abra
    3       abracadabra
    2       acadabra
    2       adabra
    1       bra
    2       bracadabra
    1       cadabra
    1       dabra
    1       ra
    2       racadabra

.. tip::

   In SeqAn there is already a function :dox:`TopDownHistoryIterator#nodeDepth` defined to return the node depth.

Additional iterators
~~~~~~~~~~~~~~~~~~~~

By now, we know the following iterators (:math:`n` = text size, :math:`\sigma` = alphabet size, :math:`d` = tree depth):

+---------------------------------+------------------------------------------+------------------------+---------------------+
| Iterator specialization         | Description                              | Space                  | Index tables        |
+=================================+==========================================+========================+=====================+
| :dox:`BottomUpIterator`         | postorder dfs                            | :math:`\mathcal{O}(d)` | SA, LCP             |
+---------------------------------+------------------------------------------+------------------------+---------------------+
| :dox:`TopDownIterator`          | can go down and go right                 | :math:`\mathcal{O}(1)` | SA, Lcp, Childtab   |
+---------------------------------+------------------------------------------+------------------------+---------------------+
| :dox:`TopDownHistoryIterator`   | can also go up, preorder/postorder dfs   | :math:`\mathcal{O}(d)` | SA, Lcp, Childtab   |
+---------------------------------+------------------------------------------+------------------------+---------------------+

Besides the iterators described above, there are some
application-specific iterators in SeqAn:

+------------------------------------+------------------------------------------+-------------+--------------------------+
| Iterator specialization            | Description                              | Space       | Index tables             |
+====================================+==========================================+=============+==========================+
| :dox:`MaxRepeatsIterator`          | maximal repeats                          | O(n)        | SA, Lcp, Bwt             |
+------------------------------------+------------------------------------------+-------------+--------------------------+
| :dox:`SuperMaxRepeatsIterator`     | supermaximal repeats                     | O(d+σ)      | SA, Lcp, Childtab, Bwt   |
+------------------------------------+------------------------------------------+-------------+--------------------------+
| :dox:`SuperMaxRepeatsFastIterator` | supermaximal repeats (optimized for ESA) | O(σ)        | SA, Lcp, Bwt             |
+------------------------------------+------------------------------------------+-------------+--------------------------+
| :dox:`MumsIterator`                | maximal unique matches                   | O(d)        | SA, Lcp, Bwt             |
+------------------------------------+------------------------------------------+-------------+--------------------------+
| :dox:`MultiMemsIterator`           | multiple maximal exact matches (w.i.p.)  | O(n)        | SA, Lcp, Bwt             |
+------------------------------------+------------------------------------------+-------------+--------------------------+

Given a string s a repeat is a substring r that occurs at 2 different positions i and j in s.
The repeat can also be identified by the triple (i,j,\|r\|).
A maximal repeat is a repeat that cannot be extended to the left or to the right, i.e. s[i-1]≠s[j-1] and s[i+\|r\|]≠s[j+\|r\|].
A supermaximal repeat r is a maximal repeat that is not part of another repeat.
Given a set of strings s1, ..., sm a MultiMEM (multiple maximal exact match) is a substring r that occurs in each sequence si at least once and cannot be extended to the left or to the right.
A MUM (maximal unique match) is a MultiMEM that occurs exactly once in each sequence.
The following examples demonstrate the usage of these iterators:

* :dox:`DemoMaximalUniqueMatches Demo Maximal Unique Matches`
* :dox:`DemoSupermaximalRepeats Demo Supermaximal Repeats`
* :dox:`DemoMaximalRepeats Demo Maximal Repeats`