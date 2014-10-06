Add a new model
===============

This chapter shows how to add a new model type to BAMM.
This new model is very simple and probably not very useful,
but it illustrates the steps required to extend BAMM.


Subclass from BranchEvent
-------------------------

Start by creating the new event class (called *MyBranchEvent*).
*MyBranchEvent* must be subclassed from the *BranchEvent* class.
For our example, *MyBranchEvent* will have just a single parameter (*alpha*)::

    #ifndef MY_BRANCH_EVENT_H
    #define MY_BRANCH_EVENT_H

    #include "BranchEvent.h"

    class Tree;
    class Node;
    class Random;

    class MyBranchEvent : public BranchEvent
    {
    public:

        MyBranchEvent(double alpha, Node* x, Tree* tp,
            Random& random, double map);
        virtual ~MyBranchEvent() {};

        void   setAlpha(double alpha);
        double alpha();

    private:

        double _alpha;
    };

    #endif

This class is easy to implement, as most of the functionality
is already in the parent class *BranchEvent*.
All that is needed is to properly initialize *BranchEvent*
and the specific parameter *alpha*::

    #include "MyBranchEvent.h"

    class Node;
    class Tree;
    class Random;

    MyBranchEvent::MyBranchEvent(double alpha, Node* x, Tree* tp,
        Random& random, double map) :
        BranchEvent(x, tp, random, map), _alpha(alpha)
    {
    }

Subclass from Model
-------------------

Declare the new model class by subclassing from *Model*.
Remember to include the virtual methods from *Model*
that must be implemented in *MyModel*::

    #ifndef MY_MODEL_H
    #define MY_MODEL_H

    #include "Model.h"

    #include <vector>
    #include <string>

    class Node;
    class Random;
    class Settings;
    class BranchEvent;

    class MyModel : public Model
    {
    public:

        MyModel(Random& rng, Settings& settings);

        virtual double computeLogLikelihood();
        virtual double computeLogPrior();

    private:

        virtual void setRootEventWithReadParameters
            (const std::vector<std::string>& parameters);
        virtual BranchEvent* newBranchEventWithReadParameters
            (Node* x, double time, const std::vector<std::string>& parameters);

        virtual BranchEvent* newBranchEventWithRandomParameters(double x);
        virtual BranchEvent* newBranchEventFromLastDeletedEvent();

        virtual void setMeanBranchParameters();
        virtual void setDeletedEventParameters(BranchEvent* be);

        double _alphaInitRoot;
        double _lastDeletedEventAlpha;
    };

    #endif

The variables ``_alphaInitRoot`` and ``_lastDeletedEventAlpha``
will be used in the implementation of *MyModel*.
The following is a simple implementation of *MyModel*::

    #include "MyModel.h"
    #include "Model.h"
    #include "Random.h"
    #include "Settings.h"
    #include "Tree.h"
    #include "Node.h"
    #include "BranchHistory.h"
    #include "BranchEvent.h"
    #include "MyBranchEvent.h"
    #include "Tools.h"

    #include <vector>
    #include <string>


    MyModel::MyModel(Random& random, Settings& settings) :
        Model(random, settings)
    {
        // Initialize root parameters
        _alphaInitRoot = _settings.get<double>("alphaInit");

        // Initialize root event
        _rootEvent =  new MyBranchEvent(_alphaInitRoot,
            _tree->getRoot(), _tree, _random, 0);
        _lastEventModified = _rootEvent;

        // Set node event of the root node
        _tree->getRoot()->getBranchHistory()->setNodeEvent(_rootEvent);

        // Initialize all branch histories to equal the root event
        forwardSetBranchHistories(_rootEvent);

        setCurrentLogLikelihood(computeLogLikelihood());

        Model::calculateUpdateWeights();
    }

    void MyModel::setRootEventWithReadParameters
        (const std::vector<std::string>& parameters)
    {
        MyBranchEvent* rootEvent = static_cast<MyBranchEvent*>(_rootEvent);
        rootEvent->setAlpha(alphaParameter(parameters));
    }

    BranchEvent* MyModel::newBranchEventWithReadParameters
        (Node* x, double time, const std::vector<std::string>& parameters)
    {
        double alpha = alphaParameter(parameters);
        return new MyBranchEvent(alpha, x, _tree, _random, time);
    }

    double MyModel::alphaParameter(const std::vector<std::string>& parameters)
    {
        return convert_string<double>(parameters[0]);
    }

    void MyModel::setMeanBranchParameters()
    {
        // See SpExModel or TraitModel for sample code
    }

    BranchEvent* MyModel::newBranchEventWithRandomParameters(double x)
    {
        double alpha = _random.uniform();
        return new MyBranchEvent(alpha, _tree->mapEventToTree(x),
            _tree, _random, x);
    }

    void MyModel::setDeletedEventParameters(BranchEvent* be)
    {
        MyBranchEvent* event = static_cast<MyBranchEvent*>(be);
        _lastDeletedEventAlpha = event->alpha();
    }

    BranchEvent* MyModel::newBranchEventFromLastDeletedEvent()
    {
        return new MyBranchEvent(_lastDeletedEventAlpha,
            _tree->mapEventToTree(_lastDeletedEventMapTime), _tree, _random,
            _lastDeletedEventMapTime);
    }

    double MyModel::computeLogLikelihood()
    {
        return _random.uniform();
    }

    double MyModel::computeLogPrior()
    {
        return _random.uniform();
    }

    double MyModel::calculateLogQRatioJump()
    {
        return 0.0;
    }

In the constructor, the initial *alpha* value for the root
is obtained as a setting,
which means the setting must be added to the Settings class (see below).
The specific root event is then created and assigned to private variables.
The rest of the code are required initialization commands.
(Eventually, some of these may be moved to the parent *Model* class.)

The *setMeanBranchParameters()* method currently does nothing,
but in your model it might need to compute something
(see *SpExModel* or *TraitModel* for sample calculations).
The *computeLogLikelihood()* and *computeLogPrior()* methods
currently return random numbers,
but in reality these should compute their real values.
The rest of the methods deal with creating the correct branch event.


Add settings to Settings
------------------------

In *MyModel*'s constructor, we initialized the root parameter
with a settings from the *Settings* class.
This setting must be known to the *Settings* class.
But before we add this setting,
we must make other changes to the *Settings* class
because a new model has been introduced.
In the *Settings* constructor,
add a new ``else if`` statement with the new model type
and call a new initialize method for it::

    // Initialize specific settings for model type
    if (modelType == "speciationextinction") {
        initializeSpeciationExtinctionSettings();
    } else if (modelType == "trait") {
        initializeTraitSettings();
    } else if (modelType == "mymodel") {
        initializeMyModelSettings();
    } else {
        exitWithErrorInvalidModelType();
    }

Next, add the method implementation to the *Settings* class
(do not forget to declare it in the header file)::

    void Settings::initializeMyModelSettings()
    {
        addParameter("alphaInit", "0.0", Required);
    }

It is in this method that we create the setting *alphaInit*.
Its default value is 0.0 and it is a required parameter.
If you would like the parameter to be normally hidden from the user,
specify ``NotRequired``.


Subclass from ModelFactory
--------------------------

As explained in the previous chapter,
a *ModelFactory* creates the specific model that is to be used.
It must be subclassed and its virtual method *createModel*
must be implemented to return the correct model type::

    #ifndef MY_MODEL_FACTORY
    #define MY_MODEL_FACTORY

    #include "ModelFactory.h"
    #include "MyModel.h"
    #include "MyModelDataWriter.h"

    class Model;
    class ModelDataWriter;

    class Random;
    class Settings;
    class Prior;

    class MyModelFactory : public ModelFactory
    {
    public:

        virtual ~MyModelFactory() {}

        virtual Model* createModel
            (Random& random, Settings& settings) const;
        virtual ModelDataWriter* createModelDataWriter
            (Settings& settings) const;
    };

    inline Model* MyModelFactory::createModel
        (Random& random, Settings& settings) const
    {
        return new MyModel(random, settings);
    }

    inline ModelDataWriter* MyModelFactory::createModelDataWriter
        (Settings& settings) const
    {
        return new MyModelDataWriter(settings);
    }

    #endif


This specific *MyModelFactory* should be created in *main.cpp*.
Update the *createModelFactory* method in *main.cpp*
to handle the new model type *modelType*::

    ...
    } else if (modelType == "mymodel") {
        log(Message) << "\nModel type: My Model\n";
        return new MyModelFactory();
    } else {
    ...


Subclass from ModelDataWriter
-----------------------------

The *MyModelFactory* class requires that a specific *ModelDataWriter*
class be created, which we call *MyModelDataWriter*::

    #ifndef MY_MODEL_DATA_WRITER_H
    #define MY_MODEL_DATA_WRITER_H

    #include "ModelDataWriter.h"

    class Settings;
    class Model;

    class MyModelDataWriter : public ModelDataWriter
    {
    public:

        MyModelDataWriter(Settings &settings);

        virtual void writeData(int generation, Model& model);
    };

    #endif

And its implementation::

    #include "MyModelDataWriter.h"
    #include "ModelDataWriter.h"

    class Settings;
    class Model;

    MyModelDataWriter::MyModelDataWriter(Settings &settings) :
        ModelDataWriter(settings)
    {
    }

    void MyModelDataWriter::writeData(int generation, Model& model)
    {
        ModelDataWriter::writeData(generation, model);
    }

It doesn't do much at the moment, but later we will add to it
in order to print out relevant information.


Initialize Tree for MyModel
---------------------------

The *Tree* class must be initialized properly for the new model.
For this simple model, the only thing that needs to be done
is to initialize the nodes appropriately as well as the tree map
in the constructor::

    ...
    } else if (settings.get("modeltype") == "mymodel") {
        setAllNodesCanHoldEvent();
        setTreeMap(getRoot());
    }
    ...


Create a control file
---------------------

At this point, you should be able to compile BAMM,
but you still need a new control file to run it.
The easy way to do that is to copy the ``divcontrol.txt`` file
from the diversification whale example to a new file
(here I call it ``mycontrol.txt``).
Then, erase any setting that is related to the diversification model
and add the *alphaInit* setting.
You should now be able to run BAMM with this control file.
However, the event data file is not written
because we have not specified which parameters to output.


Subclass from EventDataWriter
-----------------------------

The *EventDataWriter* class is responsible for writing to a file
information about each event in the tree.
It is an abstract class because some event data is specific to the model.
*EventDataWriter* handles writing out (at each generation desired)
the generation, the left and right species whose common ancestor
defines the location of the event, and the absolute time of the event.
A subclass of *EventDataWriter* needs to implement the
``specificHeader()`` method and the ``eventParameters(...)`` method.
In ``specificHeader()``, return any additional header information
for any parameters that are specific to the model.
In ``eventParameters(...)``, return the specific parameter value
from the event that is passed to it.
Below is the header file for a sample subclass of *EventDataWriter*::


    #ifndef MY_EVENT_DATA_WRITER_H
    #define MY_EVENT_DATA_WRITER_H

    #include "EventDataWriter.h"
    #include <string>

    class Settings;
    class BranchEvent;

    class MyEventDataWriter : public EventDataWriter
    {
    public:

        MyEventDataWriter(Settings& settings);
        virtual ~MyEventDataWriter();

    private:

        virtual std::string specificHeader();
        virtual std::string eventParameters(BranchEvent* event);
    };

    #endif

The implementation is as follows::

    #include "MyEventDataWriter.h"
    #include "EventDataWriter.h"
    #include "MyBranchEvent.h"

    #include <sstream>

    class Settings;
    class BranchEvent;

    MyEventDataWriter::MyEventDataWriter(Settings& settings) :
        EventDataWriter(settings)
    {
    }

    MyEventDataWriter::~MyEventDataWriter()
    {
    }

    std::string MyEventDataWriter::specificHeader()
    {
        return ",alpha";
    }


    std::string MyEventDataWriter::eventParameters(BranchEvent* event)
    {
        MyBranchEvent* specificEvent = static_cast<MyBranchEvent*>(event);

        std::ostringstream stringStream;
        stringStream << specificEvent->alpha();
        return stringStream.str();
    }

If you compile and run BAMM using the control file described above,
you will see the *event_data.txt* file being produced.
It will contain the *alpha* parameter as part of its output.
