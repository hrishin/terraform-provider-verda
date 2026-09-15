# Terraform Provider for Verda Cloud

Terraform/OpenTofu provider for managing Verda Cloud infrastructure.

## Requirements

- [Terraform](https://www.terraform.io/downloads.html) >= 1.0
- [Go](https://golang.org/doc/install) >= 1.23 (for development)
- Verda Cloud account with API credentials (Client ID and Client Secret)

## Beta version

This software is currently in beta and will contain bugs. Please try it out and report your issues in the repository.

As the provider is not yet in the Terraform registry, Terraform has to be pointed at the
binaries published on the [Releases](https://github.com/hrishin/terraform-provider-verda/releases) page.
The easiest way is a filesystem mirror, which keeps `terraform init`, version constraints and the
lock file working as usual:

```bash
./scripts/install-provider.sh --version v1.1.4   # omit --version for the latest release
export TF_CLI_CONFIG_FILE=~/.terraform.d/verda-mirror/terraformrc
```

```terraform
terraform {
  required_providers {
    verda = {
      source  = "verda-cloud/verda"
      version = "1.1.4"
    }
  }
}
```

The script downloads the zip for your platform, verifies it against the release `SHA256SUMS` and
writes the CLI config. Pass `--rc` to install it as `~/.terraformrc` instead of using
`TF_CLI_CONFIG_FILE`, or `--all` to mirror every platform.

In GitHub Actions use the bundled composite action, which runs the same script and exports
`TF_CLI_CONFIG_FILE` for the following steps:

```yaml
- uses: hashicorp/setup-terraform@v3
- uses: hrishin/terraform-provider-verda/.github/actions/setup-verda-provider@main
  with:
    version: v1.1.4 # optional, defaults to the latest release
- run: terraform init
```

Alternatively, for a locally built binary, use a development override in your `.terraformrc`:

```terraform
provider_installation {
  dev_overrides {
    "verda-cloud/verda" = "/path/to/binary/terraform-provider-verda"
  }

  direct {}
}
```

Usage instructions and examples can be found from the [examples](examples) directory.

## Using the Provider

### Installation

To use this provider, add it to your Terraform configuration:

```hcl
terraform {
  required_providers {
    verda = {
      source = "verda-cloud/verda"
    }
  }
}

provider "verda" {
  client_id     = var.verda_client_id
  client_secret = var.verda_client_secret
  # base_url    = "https://api.verda.com/v1"  # Optional, defaults to this value
}
```

### Authentication

The provider supports the following methods for providing credentials:

1. **Provider Configuration** (shown above)
2. **Environment Variables**:
   - `VERDA_CLIENT_ID`
   - `VERDA_CLIENT_SECRET`
   - `VERDA_BASE_URL` (optional)

### Resources

The provider currently supports the following resources:

#### `verda_ssh_key`

Manages SSH keys for accessing Verda instances.

```hcl
resource "verda_ssh_key" "example" {
  name       = "my-ssh-key"
  public_key = file("~/.ssh/id_rsa.pub")
}
```

#### `verda_startup_script`

Manages startup scripts that can be executed when instances are created.

```hcl
resource "verda_startup_script" "example" {
  name   = "setup-script"
  script = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y docker.io
  EOF
}
```

#### `verda_volume`

Manages storage volumes.

```hcl
resource "verda_volume" "example" {
  name     = "terraform-test-volume"
  size     = 100  # Size in GB
  type     = "NVMe"
  location = "FIN-01"
}
```

#### `verda_instance`

Manages compute instances.

```hcl
resource "verda_instance" "example" {
  instance_type = "small"
  image         = "ubuntu-22.04"
  hostname      = "my-instance"
  description   = "Example instance"
  location      = "FIN-01"

  ssh_key_ids = [verda_ssh_key.example.id]

  startup_script_id = verda_startup_script.example.id

  # Optional: Create new volumes
  volumes = [
    {
      name = "terraform-data-volume"
      size = 500
      type = "NVMe"
    }
  ]


  # Optional: Attach existing volumes
  existing_volumes = [verda_volume.example.id]
}
```

## Building the Provider

To build the provider from source:

```bash
git clone https://github.com/verda-cloud/terraform-provider-verda
cd terraform-provider-verda
go build
```

## Development

### Local Testing

To test the provider locally, you can use Terraform's development overrides. Create a `.terraformrc` file in your home directory:

```hcl
provider_installation {
  dev_overrides {
    "verda-cloud/verda" = "/path/to/terraform-provider-verda"
  }

  direct {}
}
```

Then build the provider and use it in your Terraform configurations.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

For issues and questions:
- GitHub Issues: https://github.com/verda-cloud/terraform-provider-verda/issues
- Verda Cloud Documentation: https://docs.verda.com
