# Debugging

There are two ways to debug Quick entities, both by hooking in to qb.

qb logs all queries it runs as debug logs. Configure LogBox to output debug logs for the `qb.models.Grammars.BaseGrammar` component to view them.

Additionally, qb announces a `preQBExecute` and a `postQBExectute` interception point. These interception points contain the sql and bindings being executed. You can hook in to these interception points to enable your own logging.

