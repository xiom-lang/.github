# XIOM Package Registry -- Setup Guide

> **Domain:** `https://registry.xiom-lang.com` (owned, DNS pointed to Contabo VPS)
> **Control Panel:** HestiaCP
> **Target:** Phase 5d.2 -- Production-grade package registry

## Step 1: Deploy Registry Server on VPS

The registry is a simple static JSON API. You need to serve two endpoints:

### Option A: Static JSON files (simplest, recommended)

Upload these files to the VPS under `/home/admin/web/registry.xiom-lang.com/public_html/`:

**`index.json`** -- Package index listing all published packages:
```json
{
  "packages": {
    "xiom-std": { "version": "0.47.0", "description": "XIOM Standard Library", "repo": "https://github.com/xiom-lang/std" },
    "xiom-vulkan": { "version": "0.1.0", "description": "Vulkan bindings", "repo": "https://github.com/xiom-lang/vulkan" }
  }
}
```

**`publish/`** -- POST endpoint. If using HestiaCP + PHP, create:

**`public_html/publish/index.php`:**
```php
<?php
header('Content-Type: application/json');
$body = json_decode(file_get_contents('php://input'), true);
if (!$body || !isset($body['name']) || !isset($body['version'])) {
    http_response_code(400);
    echo json_encode(['error' => 'Missing name or version']);
    exit;
}
// Append to index
$index = json_decode(file_get_contents('../index.json'), true);
$index['packages'][$body['name']] = [
    'version' => $body['version'],
    'description' => $body['description'] ?? '',
];
file_put_contents('../index.json', json_encode($index, JSON_PRETTY_PRINT));
echo json_encode(['status' => 'published', 'name' => $body['name'], 'version' => $body['version']]);
```

### Option B: Simple Go/Rust HTTP server

```go
// registry.go -- minimal package registry
package main
import ("encoding/json"; "net/http"; "os")
var index map[string]interface{}
func main() {
    data, _ := os.ReadFile("index.json")
    json.Unmarshal(data, &index)
    http.HandleFunc("/index.json", func(w http.ResponseWriter, r *http.Request) {
        json.NewEncoder(w).Encode(index)
    })
    http.HandleFunc("/publish", func(w http.ResponseWriter, r *http.Request) {
        var pkg map[string]interface{}
        json.NewDecoder(r.Body).Decode(&pkg)
        index["packages"].(map[string]interface{})[pkg["name"].(string)] = pkg
        data, _ := json.MarshalIndent(index, "", "  ")
        os.WriteFile("index.json", data, 0644)
        json.NewEncoder(w).Encode(map[string]string{"status": "published"})
    })
    http.ListenAndServe(":8080", nil)
}
```

Put behind nginx reverse proxy (HestiaCP auto-configures this for the domain).

## Step 2: SSL Certificate

In HestiaCP -> Web -> `registry.xiom-lang.com` -> Enable SSL -> Let's Encrypt.

## Step 3: Test

```bash
# From any machine with curl:
curl https://registry.xiom-lang.com/index.json
# Should return the package index

# From a XIOM project:
xiom pkg install xiom-std
# Should fetch from registry and install to vendor/
```

## Step 4: CI/CD Publish

Add to your GitHub Actions workflow:
```yaml
- name: Publish to Registry
  run: |
    curl -X POST https://registry.xiom-lang.com/publish \
      -H "Content-Type: application/json" \
      -d "{\"name\":\"${{ github.event.repository.name }}\",\"version\":\"${{ github.ref_name }}\"}"
```

## Step 5: Override for Self-Hosting

Users can override the registry URL via environment variable:
```bash
export XIOM_REGISTRY=https://my-private-registry.example.com
xiom pkg install my-package
```

## Step 6: Lockfile (Reproducible Builds)

After installing dependencies, generate the lockfile:
```bash
xiom pkg lock
# Produces xiom.lock -- commit this to version control
```

Example `xiom.lock`:
```json
{
  "package": "my-project",
  "version": "0.1.0",
  "dependencies": {
    "xiom-std": "0.47.0",
    "xiom-vulkan": "0.1.0"
  }
}
```
