# Logstash filter for ManageIQ/CloudForms

This is designed to be placed into your /etc/logstash/conf.d directory. It contains a number of grok and ruby filter plugins that attempt to pluck the
following detail out of evm.log:

* EMS refresh timings, including all of the fields that are output.
* Message dequeue timings, including an adjustment for future-dated messages (e.g. hourly and daily C&U rollups)
* The number of realtime and hourly captures on the queue.

Note that this is all solely based on data output in evm.log, and so it is dependent on when MIQ/CF outputs the requisite statistic to evm.log.

# Installing

## Input and Output

It is assumed that you have the following already configured:

* An input plugin for logstash (e.g. beats)
* An output plugin for logstash (e.g. elasticsearch)

## Tagging CloudForms logs from Filebeat

By default, the pipeline only acts on incoming logs that are tagged with the `cloudforms` tag (I've made an assumption your logs are coming from filebeat). 
It is highly recommended that you tag your MIQ/CFME logs with a dedicated tag before they reach logstash, in order to prevent running some expensive 
regular expressions over log files that will never match.

You can configure your filebeat to do just this by setting up (for example) the following prospectors in `/etc/filebeat/filebeat.yml`:

```
filebeat.prospectors:
- type: log
  enabled: true
  paths:
    - /var/www/miq/vmdb/log/evm.log

  tags: ["cloudforms"]
```

## Copying the pipeline

Copy the file to `/etc/logstash/conf.d`, or wherever your logstash pipeline configuration directory is on your logstash host.

The file `patterns/manageiq` needs to be placed somewhere your pipeline can access it, as it includes some logstash sematics necessary for grokking. By
default the pipeline will search in `/etc/logstash/patterns`, so if you want to copy this somewhere else you need to replace every occurrence of
`/etc/logstash/patterns` in the pipeline file with the path to your patterns directory.


