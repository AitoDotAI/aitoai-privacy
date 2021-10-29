# Pricing

## Subscription pricing
Aito.ai uses a subscription-based pricing model, with a one-day granularity. The unit price is determined based on your reserved capacity. The invoicing is handled on a monthly, post-payment basis. The invoice will be calculated by the 
number of started full days of subscription in the previous month. 

Discounts can be provided for pre-paid subscriptions or for longer-term reserved capacity. Contact sales@aito.ai for more details on our discount policies. 

### Invoicing and pricing granularity 
* Example 1:<br/>
For a customer starting the subscription on the 3rd of February 2018, the invoicing period will start at the start of the 4th of February, and ended at the end of the 28th of February, resulting in 25 days of usage. 
* Example 2:<br/>
If the customer starts the subscription on the 28th of February, the invoicing period is initialised on the 1st of March, and hence no invoice will be created for February, while March is fully invoiced. 

### Subscription termination
The subscription can be terminated with a one-day termination period. The invoicing period is then calculated to the end 
of the day of the termination. 

* Example: <br/>
The customer terminates the on-going subscription on the 4th of May. The invoicing period is then terminated at the end-of-the-day on the 4th. The May invoice will be generated for the days from 1st of May to the 4th of May, inclusive, resulting in 4 days of subscription for the month. 

### Capacity pricing
A reserved unit is measured by the relative amount of RAM and CPU we allocate for your environment. 

| Size of data | Capacity | Price/day |
|:------------:|:--------:|:---------:|
| < 1GB | Small | n € |
| < 10GB | Medium | k*n € |
| < 100GB | Large | m*n € |
|  100GB+ | Extra Large | Contact sales@aito.ai |

This price includes all usage of the database, up to the limits described below.

## Limits 
The limits to the service usage are set to guarantee the level of the service, both for individual users as 
well as for the collective of all users. There are soft limits, which are monitored but do not immediately 
prevent usage, and hard limits which cannot be exceeded. 

### Hard limits of the service
**CPU and RAM-constaints** are reserved, and thus in effect **hard limits**. The environment cannot use more than 
the allocated share of RAM or CPU-capacity. While the exact amount is not public information, we aim to alway guarantee sufficient capacity for the size of the data you have reserved. 

The **number of API-calls** is limited on a daily basis, as well as setting burst and rate limits for momentary use. 
#### Daily API-call limit
The **daily limit of API-calls** is set to 40.000 (40k) calls per day, per environment, corresponding roughly to 28 
requests per minute. 

The **burst limit** restricts the momentary number of calls to the API, and is set to **50 requests per second**. This 
is the highest allowed throughput within one to a few seconds. The calls are also counted against the weekly limit. 

The **rate limit**, i.e. the average requests per second for an extended period of time, is set to **10 requests per second**. The calls are counted against the weekly limit.

In summary: 

| Metric | Limit | Explanation |
|:------------:|:--------:|:---------:|
| Max calls per day | 40000 | The cumulative number of requests within a wall-clock day |
| Short time burst limit | 50 rps | The momentary max burst rate (within a few seconds) |
| Average rate limit | 10 rps | Steady rate limit, averaged over a longer period of time (minutes) |

The limits are enforced, but configurable. Contact sales if you need higher throughput. 


### Technical limits
**Payload size** is limited to 6MB per message. This includes data and all headers. For uploading larger datasets to 
the database, the _file upload API_ can be used to overcome this limit. 

**API Access limits** cannot be enforced on an IP or hostname basis. The authentication is based on API-key. The API 
is served only over HTTPS. 