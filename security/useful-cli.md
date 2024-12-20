# Useful CLI

<pre class="language-sh" data-title="InsInstall via brew for macOS"><code class="lang-sh">brew install awscli
<strong>aws --version
</strong></code></pre>

**Configure AWS CLI**

Set up credentials and default settings:

```bash
codeaws configure
```

You’ll be prompted for:

1. **Access Key ID**: Obtain it from the AWS IAM Management Console.
2. **Secret Access Key**: Provided with the access key.
3. **Default Region**: E.g., `us-east-1`.
4. **Output Format**: Choose `json`, `text`, or `table`.

