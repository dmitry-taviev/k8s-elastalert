# README #

##Testing rules locally#
* Setup elasticsearch locally: `kubectl port-forward --namespace=kube-system elasticsearch-logging-0 5555:9200`
* Create a rule `test.yaml` with correct es host & port:
```
#!yaml
es_host: localhost
es_port: 5555
name: ModelCreatorWarning
index: logstash-*
type: frequency
num_events: 2
timeframe:
  minutes: 1
filter:
- term:
    kubernetes.container_name: "clip-model-creator"
- term:
    kubernetes.namespace_name: "default"
- query:
    query_string:
      query: "log: Warning"
alert:
- "slack"
```
* Test it: `docker run --rm --network=host -v $(pwd)/rules/:/opt/rules/  "944590742144.dkr.ecr.eu-west-1.amazonaws.com/apply/smart-elastalert:latest" elastalert-test-rule /opt/rules/test.yaml`