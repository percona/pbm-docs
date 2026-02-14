# pbm-agent configuration file options

The following options are available for the `pbm-agent` configuration file. Read more about how to use the configuration file in the [Start pbm-agent using the configuration file](../manage/start-agent-with-config.md) chapter.

```yaml
mongodb-uri: mongodb://pbm:mysecret@localhost:27017/
backup:
  dump-parallel-collections: 10
log:
  path: "/var/log/pbm.json"
  level: "I"
  json: true
```

## mongodb-uri

*Type*: string <br>
*Required*: YES

The MongoDB connection string URI that the `pbm-agent` uses to connect to the local `mongod` node. This option is mandatory to start the `pbm-agent`.

## backup.dump-parallel-collections

*Type*: int <br>
*Required*: NO <br>
*Default*: number of CPU cores / 2 (minimum 1)

The number of collections to dump in parallel during a logical backup. By default, the number of parallel collections is half of the number of CPU cores, with a minimum value of 1.

## log.path

*Type*: string <br>
*Required*: NO <br>
*Default*: `/dev/stderr`

The path to the log file where a `pbm-agent` writes its logs. The file is created if it doesn't exist. The default value is `/dev/stderr`, which means that logs are written to the standard error output. If PBM cannot write logs to the specified path due to an error, it falls back to the default path.

## log.level

*Type*: string <br>
*Required*: NO <br>
*Default*: `D`

The log severity level. Supported levels are (from low to high): `D` - Debug (default), `I` - Info, `W` - Warning, `E` - Error, `F` - Fatal.

The output includes both the specified severity level and all higher ones. For example, if you set the level to `I` (Info), the output will include Info, Warning, Error, and Fatal messages.

## log.json

*Type*: boolean <br>
*Required*: NO <br>
*Default*: `false`

Output log messages in JSON format. If set to `false` or undefined, logs are written in the default text format.

