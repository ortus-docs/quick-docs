# Getting Started

{% hint style="success" %}
qb is the engine that powers Quick.  It is highly recommended that you become familiar with the [qb documentation.](https://qb.ortusbooks.com/)
{% endhint %}

## Configure a default datasource in your CFML engine

You can do this any way you'd like: through the web admin, in `Application.cfc`, or using [cfconfig](https://cfconfig.ortusbooks.com).

Make sure to set `this.datasource` in your `Application.cfc` so Quick knows which datasource to use.

{% hint style="info" %}
Quick can use [multiple datasources](defining-an-entity/#multiple-datasource-support), but it makes it easier to use when you don't have to deal with that.
{% endhint %}

## Download Quick

The easiest way to download Quick is to use ForgeBox with CommandBox. Just run the following from the root of your application:

`box install quick`

## Add a mapping for `quick` in your `Application.cfc`

For a default installation in a ColdBox template, the following line will do the trick.

`this.mappings[ "/quick" ] = COLDBOX_APP_ROOT_PATH & "/modules/quick";`

## Configure your `defaultGrammar` in `config/ColdBox.cfc`

Quick will auto discover your grammar by default on startup. To avoid this check, set a `BaseGrammar`.

`defaultGrammar` is a module setting for Quick. Set it in your `config/ColdBox.cfc` like so:

```
moduleSettings = {
    quick = {
        defaultGrammar = "MySQLGrammar@qb"
    }
};
```

Valid options are any of the [qb supported grammars](https://qb.ortusbooks.com). At the time of writing valid grammar options are: `MySQLGrammar@qb`, `PostgresGrammar@qb`, `SqlServerGrammar@qb` and `OracleGrammar@qb`. You can also have qb discover your grammar on application init using `AutoDiscover@qb`. Please check the qb docs for additional options.

If you want to use a different datasource and/or grammar for individual entitities you can do so by [adding some metadata](defining-an-entity/#multiple-datasource-support) attributes to your entities.
