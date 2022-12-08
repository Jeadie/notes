# On Observability
Notes on what is needed to understand a complex system, the open source tooling to consider, and various best practices


## Observability
How well can the internals, or inner workings, of a system be understood from external signals.


## Telemetry
Telemetry is the data (or external signals) emitted or pulled from a system (e.g. application, scaling group, cluster, etc) that lets you understand a system. Telemetry you might want:
- Logs: 
     - Application Logs: Outputs from application code. Think `print()`, `fmt.Println` or anything that looks like `log.info(...`
     - Request Logs: Outputs from an API/server with data pertaining to a single API request. Often application logs are embedded into a request log. 
        - Request logs can also refer to requests made by a server to a dependency (e.g. a database call), depending on the point of view. A dependency request log is just a client-isde log of the associated request log of the dependency (i.e. the database will have a matching request log). 
- Metrics:
    - Application Metrics: Business or logic level metrics. Relate to the inputs, decisions or decision cases of a process/request. Generally included in a request log/object.
    - Hardware Metrics: Pertaining to the performance of the hardware (e.g. EC2 instance, container, lambda process) running application code. 
    - Network Metrics: Similar to hardware metrics, but pertaining to low level metrics & performance of network components (e.g. throughput, requests queued, latency or integration latency)
- Traces: telemetry and metrics needed to relate telemetery between microservices into a unified context (e.g. a single user request goes through many microservices). Often this is in the form of trace context propagation (i.e. pass around a traceID and add to request logs).
    - Span: Logically related components within a trace (e.g. group multiprocessing, paginated API calls). 

## Instrumentation (and OpenTelemetry)
Instrumentation is how to get, process and store telemetery data.

- Collectors: A process on or adjacent to the application, responsible for collecting and sending data to observability backends.
    - Receivers: Get telemetry from application/host. E.g. Read from log file, periodically make system calls to get host metrics.
    - Processors: Anything that needs to be done before exporting: batching, encryption, compression, augmentation.
    - Exporters: Send data to observability backend.

- Observability Backends: Services to receieve, store and view telemetry.

Ideally observability backends support open telemetry protocols (OTLP).
 - Applications can just implement OTLP collectors 
 - No vendor lock-in and support multiple vendors without affecting application instrumentation.

## Promotheus
- Open source metrics backend
- Traditionally a pull based metrics backend. i.e. clients configure a prometheus compatible endpoint, and prometheus will periodically send requests.
- Can be pushed metrics, useful for short-lived jobs/processes/events, but also for opentelelmetry exporters.

