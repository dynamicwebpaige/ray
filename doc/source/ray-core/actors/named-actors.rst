Named Actors
============

An actor can be given a unique name within their :ref:`namespace <namespaces-guide>`.
This allows you to retrieve the actor from any job in the Ray cluster.
This can be useful if you cannot directly
pass the actor handle to the task that needs it, or if you are trying to
access an actor launched by another driver.
Note that the actor will still be garbage-collected if no handles to it
exist. See :ref:`actor-lifetimes` for more details.

.. tabbed:: Python

    .. code-block:: python

        # Create an actor with a name
        counter = Counter.options(name="some_name").remote()

        ...

        # Retrieve the actor later somewhere
        counter = ray.get_actor("some_name")

.. tabbed:: Java

    .. code-block:: java

        // Create an actor with a name.
        ActorHandle<Counter> counter = Ray.actor(Counter::new).setName("some_name").remote();

        ...

        // Retrieve the actor later somewhere
        Optional<ActorHandle<Counter>> counter = Ray.getActor("some_name");
        Assert.assertTrue(counter.isPresent());

.. tabbed:: C++

    .. code-block:: c++

        // Create an actor with a globally unique name
        ActorHandle<Counter> counter = ray::Actor(CreateCounter).SetGlobalName("some_name").Remote();

        ...

        // Retrieve the actor later somewhere
        boost::optional<ray::ActorHandle<Counter>> counter = ray::GetGlobalActor("some_name");

    We also support non-global named actors in C++, which means that the actor name is only valid within the job and the actor cannot be accessed from another job

    .. code-block:: c++

        // Create an actor with a job-scope-unique name
        ActorHandle<Counter> counter = ray::Actor(CreateCounter).SetName("some_name").Remote();

        ...

        // Retrieve the actor later somewhere in the same job
        boost::optional<ray::ActorHandle<Counter>> counter = ray::GetActor("some_name");

.. note::

     Named actors are only accessible in the same namespace.

.. tabbed:: Python

    .. code-block:: python

        import ray

        @ray.remote
        class Actor:
          pass

        # driver_1.py
        # Job 1 creates an actor, "orange" in the "colors" namespace.
        ray.init(address="auto", namespace="colors")
        Actor.options(name="orange", lifetime="detached")

        # driver_2.py
        # Job 2 is now connecting to a different namespace.
        ray.init(address="auto", namespace="fruit")
        # This fails because "orange" was defined in the "colors" namespace.
        ray.get_actor("orange")

        # driver_3.py
        # Job 3 connects to the original "colors" namespace
        ray.init(address="auto", namespace="colors")
        # This returns the "orange" actor we created in the first job.
        ray.get_actor("orange")

.. tabbed:: Java

    .. code-block:: java

        import ray

        class Actor {
        }

        // Driver1.java
        // Job 1 creates an actor, "orange" in the "colors" namespace.
        System.setProperty("ray.job.namespace", "colors");
        Ray.init();
        Ray.actor(Actor::new).setName("orange").remote();

        // Driver2.java
        // Job 2 is now connecting to a different namespace.
        System.setProperty("ray.job.namespace", "fruits");
        Ray.init();
        // This fails because "orange" was defined in the "colors" namespace.
        Optional<ActorHandle<Actor>> actor = Ray.getActor("orange");
        Assert.assertFalse(actor.isPresent());  // actor.isPresent() is false.

        // Driver3.java
        System.setProperty("ray.job.namespace", "colors");
        Ray.init();
        // This returns the "orange" actor we created in the first job.
        Optional<ActorHandle<Actor>> actor = Ray.getActor("orange");
        Assert.assertTrue(actor.isPresent());  // actor.isPresent() is true.


.. _actor-lifetimes:

Actor Lifetimes
---------------

.. tabbed:: Python

    Separately, actor lifetimes can be decoupled from the job, allowing an actor to
    persist even after the driver process of the job exits.

    .. code-block:: python

        counter = Counter.options(name="CounterActor", lifetime="detached").remote()

    The CounterActor will be kept alive even after the driver running above script
    exits. Therefore it is possible to run the following script in a different
    driver:

    .. code-block:: python

        counter = ray.get_actor("CounterActor")
        print(ray.get(counter.get_counter.remote()))

    Note that the lifetime option is decoupled from the name. If we only specified
    the name without specifying ``lifetime="detached"``, then the CounterActor can
    only be retrieved as long as the original driver is still running.

.. tabbed:: Java

    Customizing lifetime of an actor hasn't been implemented in Java yet.

.. tabbed:: C++

    Customizing lifetime of an actor hasn't been implemented in C++ yet.

