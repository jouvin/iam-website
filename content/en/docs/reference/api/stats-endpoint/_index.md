---
title: Statistical endpoint
---

Starting from v1.10.0 version, a statistical endpoint showing the number of users in the IAM organization was added.

The endpoint does not require authentication and is available at `/stats`. An example:

```bash
$ curl http://localhost:8080/stats --silent | jq
{
  "numberOfUsers": 255,
  "activeUsers": 254
}
```

The output may be populated with more information in later versions.
