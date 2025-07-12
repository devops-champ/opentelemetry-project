## Adding a Metrics

We will use Prometheus to fetch the metrics of our app. So, Prometheus is metrics time series database.


## Instrumenting Metrics

Run the following command to install the metrics related packages:

```
yarn add @opentelemetry/sdk-metrics @opentelemetry/exporter-prometheus
```

- `@opentelemetry/sdk-metrics`: Provides the core OpenTelemetry SDK for collecting, processing, and exporting metrics in Node.js applications.

- `@opentelemetry/exporter-prometheus`: Exports collected OpenTelemetry metrics in a format that Prometheus can scrape via an HTTP endpoint.

### Creating Prometheus Config File

```
mkdir prometheus
nano prometheus.yml
```

Add the following config in `prometheus.yml` file:
```
global:
  scrape_interval: 1s

scrape_configs:
  - job_name: 'opentelemetry'
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
    - targets: ['todo:9464','auth:9464']
``` 

- Sets the default scrape interval to 1s, meaning Prometheus will collect metrics from all configured targets every second.
- A scrape job named opentelemetry. Prometheus uses job names to group and label metrics from related targets.
- Lists static targets (i.e., fixed endpoints) where Prometheus will scrape metrics. todo:9464 and auth:9464 are service names (probably Docker container hostnames) exposing metrics on port 9464.


### Prometheus Docker Image

Add the Prometheus service into the docker-compose.yml

```
  prometheus:
    image: prom/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    volumes:
      - ./prometheus/:/etc/prometheus/
    ports:
      - 9090:9090
```      

### Configure tracer.ts

Adding the following importers:
```
import { PrometheusExporter } from '@opentelemetry/exporter-prometheus';
import { MeterProvider } from '@opentelemetry/sdk-metrics';
import { Resource } from '@opentelemetry/resources';
import { SemanticResourceAttributes } from '@opentelemetry/semantic-conventions';
```

```
    const { endpoint, port } = PrometheusExporter.DEFAULT_OPTIONS;
    const exporter = new PrometheusExporter({}, () => {
        console.log(
          `prometheus scrape endpoint: http://localhost:${port}${endpoint}`,
        );
      });
    const meterProvider = new MeterProvider({
            resource: new Resource({
            [SemanticResourceAttributes.SERVICE_NAME]: serviceName,
        }),
    });    meterProvider.addMetricReader(exporter);
    const meter = meterProvider.getMeter('my-service-meter');
```

Return the meter:
```
return meter;
```

Add the following in `todo-service.ts`:
```
const calls = meter.createHistogram('http-calls');

app.use((req,res,next)=>{
    const startTime = Date.now();
    req.on('end',()=>{
        const endTime = Date.now();
        calls.record(endTime-startTime,{
            route: req.route?.path,
            status: res.statusCode,
            method: req.method
        })
    })
    next();
})
```
