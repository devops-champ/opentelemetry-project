## opentelemetry-project

I will be documenting the skills I learned by instrumenting the application.

This is a simple To-Do application written in typescript. The scope of this project is to instrument the opentelemetry to get the application traces and spans.


## Observability

Moder applications have lots of moving pieces. Observability helps you collect the data and visualize it what's going on in your system.


## What is OpenTelemetry?

OpenTelemetry is an open-source tool that connects your application to the observability backend. Opentelemetry collects the data such as logs, traces, and metrics. These collection of data is called telemetry data.


### Installing OpenTelemetry

Add OpenTelemetry packages to your project using Yarn:

```
$ yarn add @opentelemetry/api @opentelemetry/auto-instrumentations-node @opentelemetry/exporter-trace-otlp-proto @opentelemetry/sdk-node
```

`@opentelemetry/api` Provides the core interfaces for OpenTelemetry tracing and context propagation.

`@opentelemetry/auto-instrumentations-node` Enables automatic instrumentation for popular Node.js libraries.

`@opentelemetry/exporter-trace-otlp-proto` Exports trace data using the OTLP protocol in proto format.

`@opentelemetry/sdk-node` Bundles the OpenTelemetry SDK setup for Node.js applications with sensible defaults.


create tracer.ts:
```
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-proto';

function start(serviceName: string) {
    const traceExporter = new OTLPTraceExporter({
        url: 'http://jaeger:4318/v1/traces',
    });

    const sdk = new NodeSDK({
        traceExporter,
        serviceName:serviceName,
        instrumentations: [getNodeAutoInstrumentations()]
    });


    sdk.start();
}

export default start
```
