# Mongodb



```bash
use db;

var collNames = db.getCollectionNames();

for (var i = 0; i < collNames.length; i++) {   var coll = db.getCollection(collNames[i]);    var stats = coll.stats(1024 * 1024);    print(stats.ns, stats.totalIndexSize, stats.totalSize); }
```

