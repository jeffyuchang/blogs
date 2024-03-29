--- 
layout: post
title: "Exploring ODE part V: implemenation of scheduler-simple module."
published: true
tags: 
- BPM
- Java
- ODE
- RiftSaw
type: post
status: publish
---
In Ode/RiftSaw, we make the process execution as asynchronous, which means that if you running a bpel process, we are using more than one thread to accomplish this invocation.

And this is what the scheduler-simple module for, it takes care of putting a task into database and pull tasks out of database and how to run them. Lets make a simple example here, say your bpel process has a invoke Activity to invoke an external web service. In Ode, just right before entering invoke Activity, we've created a job that captures this invoke activity information, and store it into the database. Because once we've started Ode Bpel Server, we already started a background thread that checks this ode_job table periodically, once we've found that there is a job needs to be executed, it will load it from database, put it into memory, and then submit it to the ExecutorService for execution.

In this blogpost, we will examine this module's architecture and important APIs.

First is the Task API, this is the parent class for Job and SchedulerTask.

{% highlight java %}
class Task {
    /** Scheduled date/time. */
    public long schedDate;


    Task(long schedDate) {
        this.schedDate = schedDate;
    }
}
{% endhighlight %}


It is very simple, just had a scheduled date for its execution.
Next, we will see the Job's API, Job is for invoking an external service and like. we've put all of important information into the JobDetail object.

{% highlight java %}
class Job extends Task {
    private static final SimpleDateFormat DATE_FORMAT = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss z");
    String jobId;
    boolean transacted;
    JobDetails detail;
    boolean persisted = true;

    public Job(long when, String jobId, boolean transacted, JobDetails jobDetail) {
        super(when);
        this.jobId = jobId;
        this.detail = jobDetail;
        this.transacted = transacted;
    }
....
}
{% endhighlight %}


Now, lets look at another type of Task, which is called SchedulerTask.

{% highlight java %}
private abstract class SchedulerTask extends Task implements Runnable {
    SchedulerTask(long schedDate) {
        super(schedDate);
    }
}
{% endhighlight %}


This is an abstract class, its subclasses are: LoadImmediateTask, UpgradeJobsTask, CheckStaleNodes.

To understand these tasks, it is better that we look at what SimpleScheduler class defined. In Ode, the job design was based around three time horizons: "immediate", "near future", and "everything else".
Immediate jobs (i.e. jobs that are about to be up) are written to the database and kept in an in-memory priority queue. When they execute, they are removed from the database. Near future jobs are placed in the database and assigned to the current node, however they are not stored in
memory. Periodically jobs are "upgraded" from near-future to immediate status, at which point they get loaded into memory. Jobs that are further out in time, are placed in the database without a node identifer; when they are ready to be "upgraded" to near-future jobs they are assigned to one
of the known live nodes. recovery is straight forward, with stale node identifiers being reassigned to known good nodes.

In terms of time, we defined two variables, one is: \_immediateInterval and  \_nearFutureInterval.
if a job's scheduled date is between \[now, now +  \_immediateInterval\], it belongs to the "immediate" job.
while if it is in \[now + \_immediateInterval, now +  \_nearFutureInterval\], it belongs to the "near future" job then.

You can check the SimpleScheduler.doLoadImmediate() and SimpleScheduler.doUpgrade() respectively for its logic.

Also, you may be aware that we've also had the CheckStaleNodes task, this is basically for clustering work, to
check if there are any stale nodes, if it has, we will move the assigned jobs over to other node by updating nodeId.

So now, we've seen different Tasks, like Jobs and SchedulerTask. Now, we will need an interface to run these Tasks, hence TaskRunner was introduced.

{% highlight java %}
interface TaskRunner {
    public void runTask(Task task);
}
{% endhighlight %}


Here is the implementation from SimpleScheduler.TaskRunner() method.

{% highlight java %}
public void runTask(final Task task) {
    if (task instanceof Job) {
        Job job = (Job)task;
        if( job.detail.getDetailsExt().get("runnable") != null ) {
            runPolledRunnable(job);
        } else {
            runJob(job);
        }
    } else if (task instanceof SchedulerTask) {
        _exec.submit(new Callable() {
            public Void call() throws Exception {
                try {
                    ((SchedulerTask) task).run();
                } catch (Exception ex) {
                    __log.error("Error during SchedulerTask execution", ex);
                }
                return null;
            }
        });
    }
}
{% endhighlight %}


As I said before, once we've start BpelServer, we will start a thread running, it only gets stopped only when the BpelServer is been stopped. Thats called SchedulerThread.


In this class, basically we had following members: PriorityBlockingQueue, this is queue for the immediate execution. TaskRunner, this is the
container for running Task. The logic for the running is quite straight forward.

{% highlight java %}
/**
 * Pop items off the todo queue, and send them to the task runner for processing.
 */
public void run() {
    while (!_done) {
        _lock.lock();
        try {
            long nextjob;
            while ((nextjob = nextJobTime()) > 0 &&!_done)
                _activity.await(nextjob, TimeUnit.MILLISECONDS);


            if (!_done &amp;amp;&amp;amp; nextjob == 0) {
                Task task = _todo.take();
                _taskrunner.runTask(task);

            }
        } catch (InterruptedException ex) {
            ; // ignore
        } finally {
            _lock.unlock();
        }
    }
}
{% endhighlight %}


Now that we've seen all of important APIs here, we will look at how we start SimpleScheduler when ODEServer is started.
excerpt from SimpleScheduler.start() method.

{% highlight java %}
public synchronized void start() {
    if (_running)
        return;

    if (_exec == null)
        _exec = Executors.newCachedThreadPool();

    _todo.clearTasks(UpgradeJobsTask.class);
    _todo.clearTasks(LoadImmediateTask.class);
    _todo.clearTasks(CheckStaleNodes.class);
    _processedSinceLastLoadTask.clear();
    _outstandingJobs.clear();

    _knownNodes.clear();

    try {
        execTransaction(new Callable() {

            public Void call() throws Exception {
                _knownNodes.addAll(_db.getNodeIds());
                return null;
            }

        });
    } catch (Exception ex) {
        __log.error("Error retrieving node list.", ex);
        throw new ContextException("Error retrieving node list.", ex);
    }

    long now = System.currentTimeMillis();

    // Pretend we got a heartbeat...
    for (String s : _knownNodes) _lastHeartBeat.put(s, now);

    // schedule immediate job loading for now!
    _todo.enqueue(new LoadImmediateTask(now));

    // schedule check for stale nodes, make it random so that the nodes don't overlap.
    _todo.enqueue(new CheckStaleNodes(now + randomMean(_staleInterval)));

    // do the upgrade sometime (random) in the immediate interval.
    _todo.enqueue(new UpgradeJobsTask(now + randomMean(_immediateInterval)));

    _todo.start();
    _running = true;
}
{% endhighlight %}


Also, please noted that we had two different types of JobProcessor, one is ordinary JobProcessor, the other one is PolledRunnableJobProcessor, which is meant for running some jobs that gets run periodically.
