#server.js

npm init -y

npm install @opentelemetry/api \
  @opentelemetry/sdk-trace-node \
  @opentelemetry/sdk-trace-base \
  @opentelemetry/exporter-trace-otlp-grpc \
  express
npm install @opentelemetry/exporter-trace-otlp-http

const express = require('express');
const { trace, diag, DiagConsoleLogger, DiagLogLevel } = require('@opentelemetry/api');
const { NodeTracerProvider } = require('@opentelemetry/sdk-trace-node');
const { SimpleSpanProcessor } = require('@opentelemetry/sdk-trace-base');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-http'); // ✅ Using HTTP Exporter

// Enable OpenTelemetry Debug Logging
diag.setLogger(new DiagConsoleLogger(), DiagLogLevel.ALL);

// Initialize OpenTelemetry Tracing
const provider = new NodeTracerProvider();
const exporter = new OTLPTraceExporter({
  url: 'http://localhost:4318/v1/traces', // ✅ Using HTTP endpoint for Tempo
});

// Wrap exporter with logging to debug trace sending
exporter.export = ((originalExport) => (spans, callback) => {
  console.log("🔄 Exporting spans to Tempo:");
  
  // Log only essential trace details (avoiding circular JSON issues)
  spans.forEach(span => {
    console.log(`🟢 TraceID: ${span.spanContext().traceId}, SpanID: ${span.spanContext().spanId}`);
  });

  return originalExport.call(exporter, spans, callback);
})(exporter.export);

provider.addSpanProcessor(new SimpleSpanProcessor(exporter));
provider.register();

// Create Express App
const app = express();
const tracer = trace.getTracer('tracing-app');

app.get('/trace', (req, res) => {
  console.log('✅ Received /trace GET request');

  const span = tracer.startSpan('received-get-trace');
  span.setAttribute('request.method', 'GET');
  span.setAttribute('request.url', req.url);
  span.addEvent('Processing GET request');

  setTimeout(() => {
    span.addEvent('Finished processing GET request');
    span.end();
    console.log(`✅ Tracing span ended for /trace (TraceID: ${span.spanContext().traceId}, SpanID: ${span.spanContext().spanId})`);
    res.json({ message: 'GET Trace event received!' });
  }, 500);
});

// Start the Server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`🚀 Server running at http://localhost:${PORT}`);
});
