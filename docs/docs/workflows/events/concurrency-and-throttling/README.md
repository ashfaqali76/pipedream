# Managing Concurrency and Throttling

Pipedream makes it easy to manage the concurrency and rate at which events from a source trigger your workflow code using execution controls.

## Why Is It Important?

Workflows listen for events and execute code as soon as they are triggered. While this behavior is expected for many use cases, there can be unintended consequences. 

**Concurrency**

Without restricting concurrency, events can be processed in parallel and there is no guarantee that they will execute in the order in which they were received. This can cause race conditions. 

For example, if two workflow events try to add data to Google Sheets simultaneously, they may both attempt to write data to the same row. As a result, one event can overwrite data from another event. The diagram below illustrates this example -- both `Event 1` and `Event 2` attempt to write data to Google Sheets concurrently -- as a result, they will both write to the same row and the data for one event will be overwritten and lost (and no error will be thrown). We observed this scenario resulted in data loss approximately 20% of the time with high volume workflows.

![image-20201027132901691](images/image-20201027132901691.png)

You can avoid race conditions like this by limiting workflow concurrency to a single "worker". What this means is that only one event will be processed at a time, and the next event will not start processing until the first is complete (unprocesssed events will maintained in a queue and processed by the workflow in order). The following diagram illustrates how the events in the last diagram would executed if concurrency was limited to a single worker. 

![image-20201027133308888](images/image-20201027133308888.png)

While the first example resulted in only two rows of data in Google Sheets, this time data for all three events are recorded to three separate rows.

**Throttling**

If your workflow integrates with any APIs, then you may need to limit the rate at which your workflow executes to avoid hitting rate limits from your API provider. Since event-driven workflows are stateless, you can't manage the rate of execution from within your workflow code. Pipedream's execution controls solve this problem by allowing you to control the maximum number of times a workflow may be invoked over a specific period of time (e.g., up to 1 event every second).

## How It Works

Events emitted from a source to a workflow are placed in a queue, and Pipedream triggers your workflow with events from the queue based on your concurrency and rate limit settings. These settings may be customized per workflow (so the same events may be processed at different rates by different workflows).

![image-20201027145847752](images/image-20201027145847752.png)

The maximum number of events Pipedream will queue per workflow depends on your account type.

- Up to 100 events will be queued per workflow for free and pro accounts
- Team/Enterprise accounts may have custom limits. If you need a larger queue size, please contact Pipedream

**IMPORTANT:** If the number of events emitted to a workflow exceeds the queue size, events will be lost. If that happens, an error message will be displayed in the event list of your workflow and your global error workflow will be triggered.

For more context on this feature and technical details, check out our **engineering blog post**.

## Where Do I Manage Concurrency and Throttling?

Concurency and throttling can be managed in the **Execution Controls** section of your **Workflow Settings**. Event queues are currently supported for any workflow that is triggered by an event source. Event queues are not currently supported for native workflow triggers (native HTTP, cron, SDK and email).



![image-20201027120141750](images/image-20201027120141750.png)

## Managing Event Concurency

Concurrency controls define how many events can be executed in parallel. To enforce serialized, in-order execution, limit concurency to `1` worker. This guarantees that each event will only be processed once the execution for the previous event is complete. 

To execute events in parallel, increase the number of workers (the number of workers defines the maximum number of concurrent events that may be processed), or disable concurrency controls for unlimited parallelization.

## Throttling Workflow Execution

To throttle workflow execution, enable it in your workflow settings and configure the **limit** and **interval**.

The limit defines how many events (from `0-10000`) to process in a given time period.

The interval defines the time period over which the limit will be enforced. You may specify the time period as a number of seconds, minutes or hours (ranging from `1-10000`) 

## Pausing Workflow Execution

To stop the queue from invoking your worklow, throttle workflow execution and set the limit to `0`.
