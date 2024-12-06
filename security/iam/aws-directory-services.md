# AWS Directory services

## Types

#### AWS Managed MS AD :evergreen\_tree:

* **Trust Connection with On-Premise AD**: we need to have **trust  connection with on-premise AD to 'share users' .**
* **Best for Existing On-Premise AD Users**: Ideal when your organization is already using **Microsoft AD** on-premises and wants to integrate or extend the directory to AWS. 🏢➡️☁️

#### AD Connect = proxy :motorway:

* Acts as a **proxy** to sync on-prem AD with AWS Managed Microsoft AD. It's a **bridge** between on-premise and cloud resources.
* **Two Types:**
  1. **Small (up to 500 users)**: Suited for small organizations looking to leverage AD integration with fewer user accounts. 🧑‍💻
  2. **Large (up to 5000 users)**: Ideal for corporations with larger user bases that require more extensive directory integration. 🏢📈
* AD is redirecting requests between the on-prem AD and the AWS Managed AD. Users are still managed in the on-prem AD, while **MFA** (Multi-Factor Authentication) is supported for added security. 🔐
* **Good for Solely On-Prem Managed AD Users**: If users are solely managed on-premise, this setup fits well as it integrates both local and cloud infrastructure. ☁️👨‍💻

**AWS Simple AD** 💡

* **Simple AD** is a **managed directory** that is compatible with Active Directory. It's not as feature-rich as AWS Managed MS AD, but it’s **lightweight** and designed for simpler use cases. 🎛️
* **No On-Prem Joining**: Unlike AWS Managed Microsoft AD, **Simple AD cannot be joined with an on-premises AD**, so it’s intended for standalone cloud-based setups. 🌐







