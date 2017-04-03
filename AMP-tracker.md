Our AMP tracker allows you to integrate Rakam in your AMPs. Rakam is an analytics solution provider for AMP so you can just embed the following snippet in your AMP.

```
<amp-analytics type="rakam">
<script type="application/json">
  {
    "vars": {
      "apiEndpoint": "<rakam api url>",
      "writeKey": "<key>",
    }
  }
</script>
```

In order to add more event attributes you may use `extraUrlParams`:

```
<amp-analytics type="rakam">
<script type="application/json">
  {
    "vars": {
      "apiEndpoint": "<rakam api url>",
      "writeKey": "<key>",
    },
    "extraUrlParams": {
      "prop.type": "post",
      "prop.published_at": "2017-03-28",
      "prop.author": "Emre"
    }
  }
</script>
```

If you want to customize the implementation and add track even more event, here is a more advanced use-case:

```
<amp-analytics type="rakam" id="rakam">
<script type="application/json">
{
  "vars": {
    "apiEndpoint": "<rakam api url>",
    "writeKey": "<key>",
  },
  "extraUrlParams": {
    "prop._user_name": "Emre"
  },
  "triggers": {
    "click": {
      "on": "click",
      "selector": "#test1",
      "request": "custom",
      "vars": {
        "collection": "click"
      }
    }
  }
}
</script>
</amp-analytics>
```
