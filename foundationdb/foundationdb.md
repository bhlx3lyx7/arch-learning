# FoundationDB

## Architecture
![arch](Architecture.png)

### Components
![components](all-components.jpeg)

### Workflow
Step 1, client get proxy addr
![txn-1](txn-1-get-proxy-addr.jpeg)

Step 2, client read commit version
![txn-2](txn-2-read-commit-version.jpeg)

Step 3, client read data
![txn-3](txn-3-read-data.jpeg)

Step 4, client commit txn
![txn-4](txn-4-commit-txn.jpeg)

Background periodic threads
![bg-threads](background-routine.jpeg)

## Reference
- architecture: https://apple.github.io/foundationdb/architecture.html