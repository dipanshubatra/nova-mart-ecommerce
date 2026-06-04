# Performance Testing Summary

This document summarizes the load testing results for the NovaMart e-commerce platform APIs.

## Authentication API

| Metric | Value |
|---------|---------|
| Samples | 250 |
| Average Response Time | 180-250 ms |
| Error Rate | 0% |
| Throughput | 20+ req/sec |

Load testing performed against the authentication endpoints under concurrent user traffic.

## Product Listing API

| Metric | Value |
|---------|---------|
| Samples | 5,250 |
| Average Response Time | 182 ms |
| Minimum Response Time | 110 ms |
| Maximum Response Time | 17,941 ms |
| Error Rate | 0.00% |
| Throughput | 10.7 req/sec |
| 90th Percentile | 290 ms |
| 95th Percentile | 326 ms |
| 99th Percentile | 395 ms |

Load testing performed using Apache JMeter against the public product listing endpoint under concurrent user traffic.

## Test Environment

- **Tool:** Apache JMeter
- **Test Type:** Load testing under concurrent user traffic
- **Target:** Public API endpoints
