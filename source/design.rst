Software Design
===============

This chapter describes the main classes in BAMM and how they interact.
It also explains how to extend BAMM through the inheritance of classes.


MetropolisCoupledMCMC
---------------------

This class class is responsible
for creating the desired number of Markov chains in the analysis.
It is also responsible for iterating each chain
and swapping their temperatures accordingly.
As each chain is iterated, the appropriate data writer is used
to output the specific model's data.


MCMC
----

This class is responsible for iterating a single chain
a given number of steps (or generations).
At each iteration, a new state for the model is proposed.
The acceptance ratio is obtained from the model,
and with a probability equal to the acceptance ratio,
the proposal is accepted; otherwise, it is rejected.
This process is implemented in the ``MCMC::step()`` method::

    void MCMC::step()
    {
        _model->proposeNewState();

        double acceptanceRatio = _model->acceptanceRatio();
        if (_random.trueWithProbability(acceptanceRatio)) {
            _model->acceptProposal();
        } else {
            _model->rejectProposal();
        }
    }

The *MCMC* class is also responsible for creating
the specific model to be used by the chain.
The type of the model cannot be specified directly
to the *MCMC* class because there are different model types
(e.g., diversification and phenotypic evolution).
Instead, *MCMC* relies on a *model factory* object
to create the correct model type.


ModelFactory
------------

This class is responsible for creating
the specific model (and related objects) used in the analysis.
*ModelFactory* is subclassed (it must be because it is abstract),
and its virtual method ``createModel(...)`` is implemented
in order to return the correct model type.
For example, the *SpExModelFactory* class
implements ``createModel(...)`` as follows::

    Model* SpExModelFactory::createModel
        (Random& random, Settings& settings) const
    {
        return new SpExModel(random, settings);
    }

Note that the return type of ``createModel(...)`` is a *Model*
and not a specific type of model (e.g., *TraitModel*).
This allows the *MCMC* class to treat all models the same,
without knowledge of the specific type of model at hand.

*ModelFactory*'s other virtual methods are also implemented accordingly.


Model
-----

The *Model* class represents the shift configuration
that will be updated by a Markov chain.
The *Model* class is abstract;
that is, an object of type *Model* cannot be constructed.
Instead, the *Model* class is subclassed
and its virtual methods are implemented by the subclass.
For example, the classes *SpExModel* and *TraitModel*
are subclasses of the *Model* class.
The virtual methods that are implemented are destribed below.

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
adds the specific proposals it can respond to.
Follow the principles implemented in *SpExModel* and *TraitModel*
to create a new type of model.


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
It has several pure virtual methods that are implemented by its subclasses.
*Proposal* cannot be instantiated---it is an *abstract* class.
The virtual methods and their descriptions are given below.

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
In each subclass's constructor, the ``_weight`` variable
is assigned an actual weight from the *Settings* object.

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

To create a new proposal that changes an event's parameter value,
subclass from *EventParameterProposal* and implement its virtual methods.
For an example of a class implementing these methods,
see the *LambdaInitProposal* class.
