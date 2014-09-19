Changes from original
=====================

Model
-----

The *Model* class represents the shift configuration
that will be updated by a Markov chain.
In each iteration of the chain,
a new state for the model is proposed.
The acceptance ratio is obtained from the model,
and with a probability equal to the acceptance ratio,
the proposal is accepted; otherwise, it is rejected.
The method ``MCMC::step()`` implements this clearly::

    _model->proposeNewState();

    double acceptanceRatio = _model->acceptanceRatio();
    if (_random.trueWithProbability(acceptanceRatio)) {
        _model->acceptProposal();
    } else {
        _model->rejectProposal();
    }

The *Model* class is abstract;
that is, an object of type *Model* cannot be constructed.
Instead, the *Model* class must be subclassed
and its virtual methods must be implemented by the subclass.
For example, the classes *SpExModel* and *TraitModel*
are subclasses of the *Model* class.
The virtual methods that must be implemented are destribed below.

``double computeLogLikelihood()``
    Computes and returns the log-likelihood of the model.

``double computeLogPrior()``
    Computes and returns the log-prior of the model.

``void setMeanBranchParameters()``
    Calls the appropriate methods in the *Tree* class
    to set the mean branch parameter values.

``void setRootEventWithReadParameters(...)``
    Sets the root event's parameters with the specified parameter values.

``BranchEvent* newBranchEventWithReadParameters(...)``
    Returns a new specific branch event with the specified parameter values.

``BranchEvent* newBranchEventWithRandomParameters(...)``
    Return a new specific branch event with random parameter values.

``BranchEvent* newBranchEventFromLastDeletedEvent()``
    Returns a new specific branch event with the parameter values
    equal to that of the last deleted event.

``void setDeletedEventParameters(...)``
    Saves internally the parameter values of the given branch event.

``double calculateLogQRatioJump()``
    Calculates and returns the log-q ratio jump.

Many of these methods deal with branch events
because only a specific type of model (a subclass of *Model*)
knows about a specific type of branch event (a subclass of *BranchEvent*).
For example, *SpExModel* knows about the parameters in *SpExBranchEvent*.

In addition, the constructor of the *Model* subclass
should add the specific proposals it can respond to.
Follow the principles implemented in *SpExModel* and *TraitModel*.


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


Proposal
--------

The *Proposal* class represents a proposed change to the model.
It has several pure virtual methods that must be implemented by its subclasses;
therefore, *Proposal* cannot be instantiated---it is an *abstract* class.
The methods that must be implemented and their descriptions are given below.

``void propose()``
    Changes the model object with the proposed change
    and re-calculates the new log-likelihood, saving it internally.

``void accept()``
    Sets the model's new log-likelihood to the one calculated by ``propose()``.

``void reject()``
    Undoes the changes made to the model by ``propose()``.

``double acceptanceRatio()``
    Calculates the acceptance ratio based on the model's
    log-likelihood ratio, log-prior ratio, and possibly the log-Jacobian.
    If a proposal needs to be rejected immediately for some reason,
    it should return ``0.0`` here.

The *Proposal* class also has the non-virtual method ``double weight()``,
which simply returns the value of an internal variable called ``_weight``.
This *weight* represents the update rate of the proposal.
In the subclass's constructor, the ``_weight`` variable
should be assigned an actual weight, most likely from the *Settings* object.

For an example of a class implementing these methods,
see the *EventNumberProposal* class.


EventParameterProposal
----------------------

Certain proposals simply change an event's parameter value.
If every such proposal derived directly from *Proposal*,
these subclasses would look almost identical,
except for the parameter they change.
To avoid this duplication in code,
the *EventParameterProposal* class derives from *Proposal*
and implements most of the algorithm
to change an event's parameter value.
However, this class contains virtual methods
that deal specifically with the parameter they change.
To create a new proposal that changes an event's parameter value,
subclass from *EventParameterProposal* and implement its virtual methods.
These virtual methods are described below.

``double getCurrentParameterValue()``
    Return the value of the parameter of interest from an event.

``double computeNewParameterValue()``
    Return the proposed value of the parameter.

``void setProposedParameterValue()``
    Change the event's parameter value to the proposed value.

``void reventToOldParameterValue()``
    Change the event's parameter value back to the old one
    (before the proposal).

``void updateParameterOnTree()``
    Call the appropriate methods to update the model parameters on the tree.

For an example of a class implementing these methods,
see the *LambdaInitProposal* class.
