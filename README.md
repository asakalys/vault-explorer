# vault-explorer

`vault-explorer` is a `Python` module designed to traverse the `HashiCorp Vault KV` store recursively, retrieve values, and execute specified functions on these values.

## Support

As of the current release, the module supports:

* KV Secrets Engine version 2.

## Usage

### Initialization

To begin using vault-explorer, instantiate the `VaultExplorer` class with a `hvac.Client`. 

Use the `flattenJson` parameter to control the format of the data provided to the user provided function. If `flattenJson` is set to true, the function will receive the full path to each value and the value itself. If false, the function receives the path to the secret and its JSON content.

```python
vault_url = os.getenv("VAULT_ADDR")
vault_token = os.getenv("VAULT_TOKEN")

client = hvac.Client(
    url=vault_url,
    token=vault_token
)

client.secrets.kv.v2.configure(
    mount_point = "secrets"
)

explorer = VaultExplorer(client, flattenJson=True)
```

### Application

To use `vault-explorer`, call the apply method with a path and a function. 

The function should take two arguments: `path` and `secret`. The `path` parameter will be a string representing the path to the secret, and the `secret` parameter can be either a JSON object or a string, depending on the `flattenJson` setting.

```python
explorer.apply("/", lambda path, secret: print(path, str(secret)))
```

To utilize closures for more complex operations like filtering data, define a function that captures external values and uses them within its inner function.

```python
def match(needle, matches):
    def _apply(path, secret):
        if needle in str(secret):
            matches.append((path, secret))
    return _apply

matches = []

explorer.apply("/inner/path/", match("hello world", matches))
```