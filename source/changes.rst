Changes from original
=====================

BranchEvent
-----------

The *BranchEvent* class represents the start of a process,
be it related to diversification or phenotypic evolution.
It keeps track of its location on the tree,
as well as provide methods to move itself around the tree.
Its subclasses, *SpExBranchEvent* and *TraitBranchEvent*
keep track of model-specific parameters.
For example, *TraitBranchEvent* keeps track of the initial beta,
the beta shift (rate parameter), and whether it is time-variable.
To create a new kind of branch event,
create a new class derived from *BranchEvent*
and add the parameters it should keep track of.
