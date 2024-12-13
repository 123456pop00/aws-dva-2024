---
icon: barcode-scan
---

# DDB - Scans

<figure><img src="../../../.gitbook/assets/DDB-scan-features.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/DDB-scan-example.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/DDB-scan-RCU.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/DDB-paginate--debug.png" alt=""><figcaption></figcaption></figure>

## Parallel Scan :heart\_eyes\_cat: :watermelon: workers === segments

LastEvaluatedKey (LEK) -> ExclusiveStartKey (ESK ) -> scans until LEK is null. LEK = null, means no more data &#x20;

<figure><img src="../../../.gitbook/assets/DDB-para-scan.png" alt=""><figcaption></figcaption></figure>

