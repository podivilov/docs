---
editable: false
---

# Pricing for {{ sf-name }}

## What goes into the cost of using {{ sf-name }} {#rules}

In {{ sf-name }}, you're billed for the number of function calls, computing resources allocated for the function, and outgoing traffic.

When billing computing resources (GB × hour), the memory allocated for the function and function execution time are taken into account:
* The amount of memory specified when [creating a version](operations/function/version-manage.md#func-version-create), in GB.
* The execution time for each function call in hours, rounded up to the nearest multiple of 100 ms.

{% note warning %}

You're charged for all the [function calls](concepts/function-invoke.md) that trigger your code to run.

{% endnote %}




### Pricing formula

Monthly cost = $0.043760 × Memory (GB) × Call processing time (Hours) + $0.128000 × Million calls

{% include [not-charged-functions.md](../_includes/pricing/price-formula/not-charged-functions.md) %}

{% include [free-tier.md](../_includes/pricing/price-formula/free-tier.md) %}

### Example of cost calculation {#price-example}

Example of calculating the cost of a function:
* **Memory specified when creating the version:** 512 MB.
* **Number of functions invoked:** 10000000.
* **Execution time of each call:** 800 ms.

Function cost calculation:

> 0.043760 × ((512 / 1024) × (800 / 3600 / 1000) × 10000000 -10) + 0.128000 × ((10000000 – 1000000) / 1000000)
> 
> Total: $49.336622

Where:
* 0.043760 is the price for 1 GB × hour.
* 512 / 1024 converts MB to GB, since execution time is calculated in GB×hour.
* 800 / 3600 / 1000 converts milliseconds to hours, since execution time is calculated in GB×hour.
* 10000000 is the number of function calls.
* 10 is subtracted because the first 10 GB x hour are free.
* 0.128000 is the price of 1 million function calls.
* 10000000 is the number of function calls.
* 1000000 is subtracted because the first million calls are free.
* 1000000 is the divisor used to calculate the number of millions of function calls.


## Using triggers {#triggers}

[Triggers](concepts/trigger/index.md) can be used free of charge. You can create and use triggers within the available [quotas and limits](concepts/limits.md).

## Pricing {#prices}

### Invoking a function {#invoke}




{% include [usd.md](../_pricing/functions/usd-invocations.md) %}



You pay for the actual number of invocations. The cost of a thousand calls, for instance, will be `$0.000128` at `$0.128000` for 1 million calls.


### Function execution time {#execution}




{% include [usd.md](../_pricing/functions/usd-compute.md) %}


### Outgoing traffic {#prices-traffic}




{% include notitle [usd-egress-traffic.md](../_pricing/usd-egress-traffic.md) %}
