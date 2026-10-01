# load

Measure behavior under concurrent users: throughput, latency percentiles,
error rate, and where it breaks. Run *after* correctness is proven.

## When to use

- An endpoint or job that will take real concurrent traffic
- Before a launch, after a traffic-model change, or before scaling down
- Verifying an SLO (e.g. p95 < 300ms at 200 rps)
- Finding the saturation point / where latency knees upward
- Soak: memory leaks over hours

## When NOT to use

- Correctness is not yet established → prove it with `unit`/`integration` first
- You can't isolate the environment → results will be meaningless
- As a merge gate on every commit → slow and noisy. Run on demand / nightly.

## Types

| Type | Shape | Answers |
|---|---|---|
| Smoke | 1-2 VUs, minutes | Does it work at all? |
| Load | expected peak, ~10-30 min | Does it meet the SLO? |
| Stress | ramp past peak until failure | Where does it break? |
| Spike | sudden jump | Does it recover? |
| Soak | sustained, hours | Leaks, drift, connection exhaustion |
| Breakpoint | step ramp | The actual saturation point |

## Always report percentiles, never the mean

`p50`, `p95`, `p99`, and error rate. A mean of 50ms with p99 = 4s is a broken
service that the mean hides.

## Skeleton — k6

```js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 50 },   // ramp
    { duration: '5m', target: 200 },  // hold at expected peak
    { duration: '1m', target: 0 },    // ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<300', 'p(99)<800'],
    http_req_failed: ['rate<0.01'],   // <1% errors
  },
};

export default function () {
  const res = http.get(`${__ENV.BASE}/api/users/42`);
  check(res, { 'status 200': (r) => r.status === 200 });
  sleep(1);
}
```

```bash
k6 run --env BASE=https://staging.example.com load.js
```

## Skeleton — locust (Python)

```python
from locust import HttpUser, task, between

class ApiUser(HttpUser):
    wait_time = between(1, 3)

    @task(3)
    def list_users(self):
        self.client.get("/api/users")

    @task(1)
    def get_user(self):
        self.client.get("/api/users/42")
```

```bash
locust -f locustfile.py --host https://staging.example.com --headless -u 200 -r 20 -t 5m
```

## Rules

- Never load-test production without explicit sign-off and a kill switch.
- Isolate: dedicated staging or a scaled-down replica. Note the hardware.
- Model realistic traffic: think-time, auth flow, mixed read/write ratio.
- Warm up before measuring (JIT, connection pools, caches).
- Correlate with server metrics (CPU, GC, DB connections, pool saturation).
- Record: version, hardware, config, dataset size. Without them the numbers are unusable.
- Guard with a kill switch: abort if error rate exceeds a threshold.

## Commands

| Stack | Tool | Run |
|---|---|---|
| Any HTTP | k6 | `k6 run load.js` |
| Python | locust | `locust -f locustfile.py --headless -u 200` |
| Go | vegeta | `vegeta attack -rate=200 -duration=5m < targets.txt` |
| JS | autocannon | `autocannon -c 200 -d 30 URL` |
| DB only | pgbench | `pgbench -c 32 -j 8 -T 300 mydb` |

## Pitfalls

- Load tester on the same box as the target → you measure the tester.
- Reporting averages → hides the tail that users actually feel.
- Testing an unseeded empty DB → unrealistically fast. Use production-shaped data volume.
- No warmup → first-minute numbers are garbage.
- Load test against prod pointing at real payment/email integrations. Stub them.
- Treating a pass as permanent: load results expire with every code change.
