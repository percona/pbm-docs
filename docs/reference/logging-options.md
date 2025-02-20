# Logging options

The following options are for the pbm-agent configuration. Read more about it in the [Start pbm-agent using the configuration file](../manage/start-agent-with-config.md) chapter.

```yaml
log:
  path: "/var/log/pbm.json"
  level: "info"
  json: true
```

### path

*Type*: string

The path to the log file. The file is created if it doesn't exist. The default value is `/dev/stderr` which means that the logs are written to the standard error output. If PBM cannot write logs to the specified path for due to some error, it falls back to the default path.

### level

*Type*: string

The log severity level. Supported levels are (from low to high): D - Debug (default), I - Info, W - Warning, E - Error, F - Fatal.

The output includes both the specified severity level and all higher ones.

### json

*Type*: boolean

Output log messages in JSON format. If undefined, logs are written in the default text format.