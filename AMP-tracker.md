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

If you want to customize the implementation and add track more event, here is a more advanced use-case:

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
