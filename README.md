# Logstash-to-log4net
Configuration to use log4net and logstash

### Use this conversion pattern log4net to use logstash configuration below:
  ```<param name="ConversionPattern" value="%-level [SystemName] [%d][%t] [%property{log4net:HostName}] - %message - [%exception]endline%newline" />```

### Use this contents of the Logstash configuration file:
```
  input {
      file {
        path => "path-content-log"
        start_position => "end"
        type => "logs"
        codec => multiline {
          pattern => "endline"
          negate => true
          what => next
      }
  }
  filter {
    grok {
      match => { "message" => "(?m)%{LOGLEVEL:level} \[%{DATA:application}\] \[%{TIMESTAMP_ISO8601:time}\]\[%{DATA:thread}\] \[%{DATA:hostApplication}\] - %{DATA:logMessage} - \[%{DATA:exception}: %{GREEDYDATA:logTrace}\]endline" }
      add_field => {
        "type" => "Exception"
      }
    }
    grok {
      match => { "message" => "(?m)%{LOGLEVEL:level} \[%{DATA:application}\] \[%{TIMESTAMP_ISO8601:time}\]\[%{DATA:thread}\] \[%{DATA:hostApplication}\] - %{DATA:logMessage} - \[\]endline" }
      add_field => {
        "type" => "Debug"
      }
    }
  }
  output {
    elasticsearch {
      hosts => ["url-elasticsearch"]
      user => "elastic"
      password => "password-generate-xpack"
    }
  }
  ```
