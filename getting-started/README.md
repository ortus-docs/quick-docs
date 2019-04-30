# Getting Started

## Configure a default datasource in your CFML engine

You can do this any way you'd like: through the web admin, in `Application.cfc`, or using [cfconfig](https://cfconfig.ortusbooks.com).

Make sure to set `this.datasource` in your `Application.cfc` so Quick knows which datasource to use.

## Add a mapping for `quick` in your `Application.cfc`

For a default installation in a ColdBox template, the following line will do the trick.

`this.mappings[ "/quick" ] = COLDBOX_APP_ROOT_PATH & "/modules/quick";`

## Configure your `defaultGrammar` in `config/ColdBox.cfc`

Quick will auto discover your grammar by default on startup. To avoid this check, set a `BaseGrammar`.

`BaseGrammar` is a module setting for Quick. Set it in your `config/ColdBox.cfc` like so:

```text
moduleSettings = {
    quick = {
        defaultGrammar = "MySQLGrammar"
    }
};
```

Valid options are any of the [qb supported grammars](https://qb.ortusbooks.com). At the time of writing valid grammar options are: MySQLGrammar, PostgresGrammar, MSSQLGrammar and OracleGrammar. Please check the qb docs for additional options.

If you want to use a different datasource and/or grammar for individual entitities you can do so by [adding some metadata](defining-an-entity.md#multiple-datasource-support) attributes to your entities.

