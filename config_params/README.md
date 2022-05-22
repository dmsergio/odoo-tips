## Config params

### Workers
- 1 worker ~= 6 concurrent users
- Rule of thumb: #workers = (#CPU * 2) + 1
- Example of #CPU with an Odoo instance with 60 concurrent users:
  - 60 concurrent users / 6 = 10 workers
  - CPU ~= 5
  - (5 * 2) + 1 = 11

**limit_request (8192 by default)**
- A worker will be terminated after having processed this many requests.

**limit_memory_hard (Odoo recommends 4GB)** (Set values in bytes. 1GB = 1024 MB = 1.048.576 KB = 1.073.741.824 B)
- Maximum amount of RAM a worker will be able to allocate. If a worker exceeds, Odoo will kill it immediately.

**limit_memory_soft** (Set values in bytes. 1GB = 1024 MB = 1.048.576 KB = 1.073.741.824 B)
- If a worker ends up consuming more than this limit, it will be terminated after it finishes processing the current request.

**limit_time_cpu** (Set values in seconds)
- The maximum amount of CPY time allowed to process a request.

**limit_time_real** (Set values in seconds)
- The maximum amount of wall-clock time allowed to process a request. Differs from limit_time_cpu in that this is a "wall time" limit including SQL queries.

### Logs

**logrotate = True**
- This will cause Odoo to configure the logging module to archive the server logs on a daily bases, and to keep the old logs for 30 days.
