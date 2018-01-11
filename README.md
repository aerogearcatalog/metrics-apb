# Metrics APB

[![](https://img.shields.io/docker/automated/jrottenberg/ffmpeg.svg)](https://hub.docker.com/r/aerogearcatalog/metrics-apb/)
[![Docker Stars](https://img.shields.io/docker/stars/aerogearcatalog/metrics-apb.svg)](https://registry.hub.docker.com/v2/repositories/aerogearcatalog/metrics-apb/stars/count/)
[![Docker pulls](https://img.shields.io/docker/pulls/aerogearcatalog/metrics-apb.svg)](https://registry.hub.docker.com/v2/repositories/aerogearcatalog/metrics-apb/)
[![License](https://img.shields.io/:license-Apache2-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0)

## Testing

If you're adding a new changes want to  can easily test your changes by simply running `apb test`
The test does the following:
1. Builds the APB 
1. Creates the project in currently targetted OpenShift instance
1. Runs the `provision` role
1. Runs the `test` role which checks that
    * Grafana and Prometheus routes are accessible
    * we're getting successful response from OpenShift auth proxy server

If the test ends successfully, you should see the message `Pod phase Succeeded without returning test results` in your console output.
