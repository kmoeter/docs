.. DalmatinerDB tradeoff documentation, created by
   Heinz N. Gies on Sat Jul  5 16:49:03 2014.

Tradeoffs & Design
==================

An essential part of every database is to make a decision aobut which tradeoff to make, every database makes them in one way or another and I feel it important to be upfront about them. Being open about what is won and what is lost makes it not only honesty but makes it easyer for users to make a informed decision when picking a database for their usecase.

.. note::

   Please keep in mind that all systems have tradeoffs and that just because some are not open about them does not mean they do not make them.

Foundation
----------

The design decisions for DalmatinerDB are based on a number of observation about metrics. If those observations hold true for you then chances are good that DalmatinerDB is a good fit. If they don't another system with different tradeoffs might be a better choice.

Metrics are imutable
````````````````````
Once a metric is submitted it isn't going to change any more, the CPU usage last mondah at 5:31 will not suddenly spike today. It might however happen that writing the metric is delayed, so writing in the 'past' can happen.

A single value doens't matter
`````````````````````````````
It is more important to have the bulk of data then have a single value. Usually all views are aggregated and missing values can be interpolated.

Everything is a integer
```````````````````````
Every metric is either a integer value or can be represented as one (or mutliple). Allowing to scale metrics helps here. As an example ``1.5s`` can be represented as ``1500ms``, a set of percentiles can be represented as multiple metrics (``metric.99``, ``metric.95`` ...)


Design
------

CAP
```
The perhaps first decision to make is either to pick Consistency or Availability. With metrics and the notion of immutabilit ythere is little harm in picking Availability here, so DalmatinerDB will stay available for read and write options even in the event of a network partition at the cost of giving stall reads on both sides of the pertition until it is healed (given side A can't knoow what was written at side B).

Given the immutability of metrics it can be argued that it is impossible to generate conflicting values on both sides of a split, thus merging is simple and lossless.

Filesystem
``````````
DalmatinerDB is designed to run on ZFS and other filesystems are strongly discuraged. While DalmatinerDB will start on any filesystem the experience will be greatly degraded without ZFS or a equally capable filesystem as a base.

DalmatinerDB's performance relies heaviley on taking advantage of facilities like ARC, ZIL, checksums and volume compression. Expecting those things to be handled on a filesystem level makes it possible to remove most of the code for caching, compression, validation from the application improving code simplicity, stability and performance significantly.

