=================
ecell3-python2cpp
=================

What's this?
------------

This small program translates an E-Cell 3 model written in Python to the equivalent C++ code.

The source python code::

    def buildModel( theSimulator ): 
        stepper_DE1 = theSimulator.createStepper("FixedODE1Stepper", "DE1")
        theSimulator.rootSystem.StepperID = "DE1"
        Variable___SIZE = theSimulator.createEntity("Variable", "Variable:/:SIZE")
        Variable___SIZE.Value = 1e-18
        Variable___S = theSimulator.createEntity("Variable", "Variable:/:S")
        Variable___S.Value = 1000000.0
        Variable___P = theSimulator.createEntity("Variable", "Variable:/:P")
        Variable___P.Value = 0.0
        Variable___E = theSimulator.createEntity("Variable", "Variable:/:E")
        Variable___E.Value = 1000.0
        Process___E = theSimulator.createEntity("MichaelisUniUniFluxProcess", "Process:/:E")
        Process___E.VariableReferenceList = [["S0", ":.:S", "-1"], ["P0", ":.:P", "1"], ["C0", ":.:E", "0"]]
        Process___E.KmS = "1"
        Process___E.KcF = "10"
    
    buildModel( theSimulator )
    
The resulting C++ code::

    #include <boost/scoped_ptr.hpp>
    #include <libecs/libecs.hpp>
    #include <libecs/Model.hpp>
    #include <libecs/System.hpp>
    #include <libecs/Process.hpp>
    #include <libecs/Variable.hpp>
    #include <libecs/Stepper.hpp>
    #include <iostream>
    
    void buildModel(libecs::Model& model)
    {
        libecs::Stepper* stepper_DE1 = model.createStepper("FixedODE1Stepper", "DE1");
        {
        }
        libecs::System* root = model.getRootSystem();
        {
            root->loadProperty("StepperID", libecs::Polymorph("DE1"));
        }
        libecs::Variable* P = reinterpret_cast<libecs::Variable*>(model.createEntity("Variable", libecs::FullID("Variable:/:P")));
        {
            P->loadProperty("Value", libecs::Polymorph(static_cast<libecs::Real>(0.0)));
        }
        libecs::Variable* S = reinterpret_cast<libecs::Variable*>(model.createEntity("Variable", libecs::FullID("Variable:/:S")));
        {
            S->loadProperty("Value", libecs::Polymorph(static_cast<libecs::Real>(1000000.0)));
        }
        libecs::Variable* v_E = reinterpret_cast<libecs::Variable*>(model.createEntity("Variable", libecs::FullID("Variable:/:E")));
        {
            v_E->loadProperty("Value", libecs::Polymorph(static_cast<libecs::Real>(1000.0)));
        }
        libecs::Variable* SIZE = reinterpret_cast<libecs::Variable*>(model.createEntity("Variable", libecs::FullID("Variable:/:SIZE")));
        {
            SIZE->loadProperty("Value", libecs::Polymorph(static_cast<libecs::Real>(1e-18)));
        }
        libecs::Process* p_E = reinterpret_cast<libecs::Process*>(model.createEntity("MichaelisUniUniFluxProcess", libecs::FullID("Process:/:E")));
        {
            p_E->loadProperty("KcF", libecs::Polymorph("10"));
            p_E->loadProperty("KmS", libecs::Polymorph("1"));
            p_E->registerVariableReference("S0", libecs::FullID(":.:S"), -1, false);
            p_E->registerVariableReference("P0", libecs::FullID(":.:P"), 1, false);
            p_E->registerVariableReference("C0", libecs::FullID(":.:E"), 0, false);
        }
    }
    
    int main()
    {
        typedef ModuleMaker<libecs::EcsObject> PropertiedObjectMaker;
        struct RAII
        {
            RAII() { libecs::initialize(); }
            ~RAII() { libecs::finalize(); }
        } _;
        boost::scoped_ptr<PropertiedObjectMaker> propertiedObjectMaker(libecs::createDefaultModuleMaker());
        libecs::Model model(*propertiedObjectMaker);
        
        // initialize the model
        {
            const char* dmpath = getenv("ECELL3_DM_PATH");
            if ( dmpath )
                model.setDMSearchPath(dmpath);
        }
        model.setup();
        
        // build the model
        buildModel(model);
        
        // initialize the simulation state
        model.initialize();
        
        // advance the simulation
        {
            libecs::Variable* const SIZE = reinterpret_cast<libecs::Variable*>(model.getEntity(libecs::FullID("Variable:/:SIZE")));
            libecs::Variable* const S = reinterpret_cast<libecs::Variable*>(model.getEntity(libecs::FullID("Variable:/:S")));
            libecs::Variable* const P = reinterpret_cast<libecs::Variable*>(model.getEntity(libecs::FullID("Variable:/:P")));
            libecs::Variable* const v_E = reinterpret_cast<libecs::Variable*>(model.getEntity(libecs::FullID("Variable:/:E")));
            for (int i = 0; i < 10; ++i)
            {
                std::cout << "<<time: " << model.getCurrentTime() << ">>" << std::endl;
                std::cout << "Variable:/:SIZE" << "=" << SIZE->getValue() << std::endl;
                std::cout << "Variable:/:S" << "=" << S->getValue() << std::endl;
                std::cout << "Variable:/:P" << "=" << P->getValue() << std::endl;
                std::cout << "Variable:/:E" << "=" << v_E->getValue() << std::endl;
                model.step();
            }
        }
    }

Usage
-----

::

    $ ecell3-python ecell3-python2cpp simple.py

