Rakam Pixel tracker is mostly used for embedding tracker in emails and websites which don't allow JS. 
Our AMP tracker is also based on Pixel tracker and you can also use this method when tracking data from platforms where our JS SDKs can't be used.

```
[API]/event/pixel?api.api_key=[WRITE_KEY]&collection=[COLLECTION]&prop.prop1=value1&prop.prop2=value2
```

The first two parameters are required. You can embed your properties in query parameters with `prop.` prefix.
