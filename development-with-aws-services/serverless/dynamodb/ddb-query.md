---
icon: question
---

# DDB - Query

## Query Result

* Contains  **ScanCount** (keycondition-expression item count returned ) and **Count** ( number of items in the response result if you used Filter) -> must be under 1MB or will return **partial count**



<figure><img src="../../../.gitbook/assets/Screenshot 2024-12-13 at 23.54.30.png" alt="DDB-query"><figcaption></figcaption></figure>

### Filter Expression

* can't contain PK or SK
* does not consume RCUs
* can use same conditional expression as in **keyconditon-expression**

