## What is a Tuple?

A **tuple** is a Python data structure used to store multiple values in one variable.

```python
metric = ("api-service", "GET /orders", 200, 123.5)
```

This tuple contains:

```python
service_name = "api-service"
endpoint = "GET /orders"
status_code = 200
response_time_ms = 123.5
```

A tuple looks like a list, but it uses **parentheses `()`** instead of square brackets `[]`.

```python
my_tuple = (1, 2, 3)
my_list = [1, 2, 3]
```

The big difference is:

> **Tuples are immutable.**  
> Once created, you cannot change their values.

---

## Why Tuples Exist

Tuples are useful when you want to group related values together and make sure they do not change accidentally.

Example from monitoring:

```python
cpu_metric = ("server-01", "cpu_usage", 82.4)
```

This means:

```text
server-01 has cpu_usage = 82.4
```

You probably do not want someone accidentally changing `"cpu_usage"` to `"memory_usage"` inside the same metric record.

---

## Tuple Example

```python
server_status = ("server-01", "healthy", 23.7)

print(server_status)
```

Output:

```text
('server-01', 'healthy', 23.7)
```

You can access values by index:

```python
print(server_status[0])  # server-01
print(server_status[1])  # healthy
print(server_status[2])  # 23.7
```

Indexes start from `0`.

---

## Tuple Unpacking

Tuple unpacking is one of the cleanest features.

Instead of doing this:

```python
server = server_status[0]
status = server_status[1]
latency = server_status[2]
```

You can do this:

```python
server, status, latency = server_status

print(server)
print(status)
print(latency)
```

This is very common in real code.

Monitoring example:

```python
metric = ("payment-service", "response_time_ms", 245)

service, metric_name, value = metric

print(f"{service} reported {metric_name} = {value}")
```

Output:

```text
payment-service reported response_time_ms = 245
```

---

## Tuples Are Immutable

You cannot change a tuple value after creating it.

```python
metric = ("api-service", "requests_total", 1500)

metric[2] = 2000
```

This will raise an error:

```text
TypeError: 'tuple' object does not support item assignment
```

Why?

Because tuples are designed to represent fixed data.

---

## Tuple vs List

Use a **tuple** when the structure should not change.

Use a **list** when you need to add, remove, or update items.

```python
# Tuple: fixed metric record
metric = ("auth-service", "error_rate", 0.03)

# List: collection of many metrics
metrics = [
    ("auth-service", "error_rate", 0.03),
    ("payment-service", "latency_ms", 210),
    ("order-service", "cpu_usage", 74.5),
]
```

Here, each metric is a tuple because each record has a fixed structure.

But `metrics` is a list because we may add more metrics later.

```python
metrics.append(("notification-service", "memory_usage", 61.2))
```

---

## Real Monitoring Example

Imagine you collect health checks from multiple services:

```python
checks = [
    ("auth-service", "healthy", 120),
    ("payment-service", "slow", 850),
    ("order-service", "healthy", 95),
    ("notification-service", "down", None),
]
```

Each tuple contains:

```text
(service_name, status, response_time_ms)
```

Now we can process them:

```python
for service, status, response_time in checks:
    if status == "down":
        print(f"ALERT: {service} is down")
    elif response_time and response_time > 500:
        print(f"WARNING: {service} is slow: {response_time}ms")
    else:
        print(f"OK: {service}")
```

Output:

```text
OK: auth-service
WARNING: payment-service is slow: 850ms
OK: order-service
ALERT: notification-service is down
```

This is a very practical use case for tuples.

---

## Tuple with One Item

Be careful here.

This is **not** a tuple:

```python
x = ("cpu_usage")
```

This is just a string.

To create a tuple with one item, you need a comma:

```python
x = ("cpu_usage",)
```

Check it:

```python
print(type(("cpu_usage")))
print(type(("cpu_usage",)))
```

Output:

```text
<class 'str'>
<class 'tuple'>
```

The comma is what makes it a tuple.

---

## Tuples Can Be Used as Dictionary Keys

Because tuples are immutable, they can often be used as dictionary keys.

Monitoring example:

```python
error_counts = {
    ("auth-service", 500): 12,
    ("auth-service", 401): 87,
    ("payment-service", 503): 5,
}
```

Access value:

```python
print(error_counts[("auth-service", 500)])
```

Output:

```text
12
```

This is useful when you want to group data by multiple dimensions.

For example:

```text
(service_name, status_code) -> count
```

Another example:

```python
latency_by_endpoint = {
    ("api-service", "GET /orders"): 120,
    ("api-service", "POST /orders"): 210,
    ("payment-service", "POST /pay"): 430,
}
```

This is common in monitoring, metrics, aggregation, and analytics logic.

---

## Tuple Methods

Tuples have only a few methods because they are immutable.

### `count()`

Counts how many times a value appears.

```python
statuses = ("healthy", "healthy", "down", "slow", "healthy")

print(statuses.count("healthy"))
```

Output:

```text
3
```

### `index()`

Returns the first position of a value.

```python
statuses = ("healthy", "slow", "down")

print(statuses.index("down"))
```

Output:

```text
2
```

---

## Nested Tuples

A tuple can contain other tuples.

```python
server_metrics = (
    ("cpu_usage", 82.5),
    ("memory_usage", 68.2),
    ("disk_usage", 91.0),
)
```

Loop over them:

```python
for metric_name, value in server_metrics:
    print(metric_name, value)
```

Output:

```text
cpu_usage 82.5
memory_usage 68.2
disk_usage 91.0
```

---

## Senior Insight

Tuples are great for **small, fixed-shape data**.

Good use:

```python
(service_name, metric_name, value)
```

Good use:

```python
(host, port)
```

Good use:

```python
(status_code, response_time_ms)
```

But if the tuple becomes too large, the code becomes hard to read.

Bad example:

```python
event = ("api-service", "GET /orders", 200, 123.5, "us-east-1", "trace-123", True)
```

What does `event[5]` mean? It is not clear.

In that case, use `NamedTuple`, `dataclass`, or a normal class.

---

## Better Alternative: NamedTuple

A `NamedTuple` gives tuple behavior but with readable field names.

```python
from typing import NamedTuple


class Metric(NamedTuple):
    service: str
    name: str
    value: float


metric = Metric("payment-service", "latency_ms", 245)

print(metric.service)
print(metric.name)
print(metric.value)
```

Output:

```text
payment-service
latency_ms
245
```

This is much clearer than:

```python
metric[0]
metric[1]
metric[2]
```

---

## Practical Rule

Use tuples when:

```text
The data has a fixed structure and should not be changed.
```

Use lists when:

```text
You need to add, remove, or update items.
```

Use `NamedTuple` or `dataclass` when:

```text
The data has many fields and readability matters.
```

---

## Final Example

```python
from typing import NamedTuple


class HealthCheck(NamedTuple):
    service: str
    status: str
    latency_ms: int | None


checks = [
    HealthCheck("auth-service", "healthy", 120),
    HealthCheck("payment-service", "slow", 850),
    HealthCheck("notification-service", "down", None),
]


for check in checks:
    if check.status == "down":
        print(f"ALERT: {check.service} is down")
    elif check.latency_ms and check.latency_ms > 500:
        print(f"WARNING: {check.service} is slow: {check.latency_ms}ms")
    else:
        print(f"OK: {check.service}")
```

This is a clean, real-world way to use tuple-like data in monitoring systems.