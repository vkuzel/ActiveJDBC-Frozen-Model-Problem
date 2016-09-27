ActiveJDBC + Gradle3 Daemon = Frozen Model
==========================================

This project is a small demonstration of a "class is frozen" problem running ActiveJDBC instrumentation from a Gradle daemon.
It also shows some proposed solutions to this. Check out the build.gradle file for more details.

How to simulate this
--------------------

Just run the `gradle instrumentModel` for two times.

Problem description
-------------------

1. Gradle Daemon runs tasks in its worker thread. This thread and its resource are kept alive and reused when the task is executed again.
2. ActiveJDBC instrumentation uses javasist.ClassPool singleton. This pool caches CtClass objects. This cache is not garbage collected on task finish because of 1.
3. Instance of CtClass is a container of an instrumented class. This container has property `wasFrozen` which is set to true when instrumentation is done.
Because of 1. and 2. when task is executed for the second time the existing instance with `wasFrozen == true` is retrieved from ClassPool which causes a "frozen exception".

`> org.javalite.instrumentation.InstrumentationException: java.lang.RuntimeException: frozen_model.SomeModel class is frozen`
