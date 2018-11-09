# Quartz Scheduler Model


In this article we will brief you about the internal architecture of Quartz Scheduler. We will first review the main components of a Quartz Scheduler and then look into its model.

## Main Quartz Scheduler Components

Job and Trigger represent the ‘what’ is to be run and ‘when’ it is to be run and thus they form the main components of a Quartz Scheduler. Rest of the components are there to make sure this happens smoothly and efficiently.

## Quartz Scheduler Model

Let’s review each component from the bird’s eye view.

1.  Scheduler Factory – Scheduler factory is the one responsible to build the scheduler model, wire in all the dependent components, based on the contents of quartz properties file.
2.  Scheduler – This maintains a registry of job and trigger. It is also responsible for executing the job when their associated triggers fire.
3.  Scheduler Thread – This is the thread responsible for performing the work of firing trigger. It contacts the job store to get the next set of the triggers to be fired.
4.  Job – This is the interface that must be implemented by the task to be executed.
5.  Trigger – Triggers are the mechanism by which we schedule the jobs.They instruct the scheduler when the job should be fired.
6.  JobStore – This is the interface to be implemented by the classes that provide storage mechanism for job and trigger.
7.  ThreadPool – The job to be executed is passed the pool of threads represented by ThreadPool.

 [![Quartz Scheduler Components](http://www.javarticles.com/wp-content/uploads/2016/03/MainComponents-702x336.png)](http://www.javarticles.com/wp-content/uploads/2016/03/MainComponents.png) 

Quartz Scheduler Components

## Quartz Scheduler Internal Model

There are many more components which help configure and facilitate job execution, for example, the thread executor that decides how to execute the Quartz Scheduler Thread. The thread pool that manages the pool of threads to which the job is delegated for its execution, flavors of job store, one of which is in-memory based and the other is database specific.
Note that there can be more than one trigger pointing to the same job but a single trigger can only point to one job.

[](http://www.javarticles.com/wp-content/uploads/2016/03/QuartzSchedulerModel.png)

[![Quartz Scheduler Model](http://www.javarticles.com/wp-content/uploads/2016/03/QuartzSchedulerModel-768x438.png)](http://www.javarticles.com/wp-content/uploads/2016/03/QuartzSchedulerModel.png)

Comments are closed.

Measure

Measure