# Domain Join Automation

Clean and organized Ansible automation for joining Linux and Windows servers to Active Directory domains.

**GitHub Workflow-Driven** - Fully automated CI/CD pipeline for domain joining operations.

**Designed for Air-Gapped Environments** - No external dependencies or internet connectivity required.

## 🏗️ **Project Structure**

```bash
C:.
│   .gitignore
│   ansible.cfg
│   README.md
│   docker-compose.yml
│   Dockerfile
│
├───configs
│       project_config.yml             # 🎯 SINGLE SOURCE for all project values
│
├───inventory
│   │   hosts.ini.template             # 📝 Dynamic inventory template
│   └───group_vars                     # 📋 Group-specific variables
│
├───.github/workflows
│       domain_join.yml                # 🚀 GitHub Actions workflow
│
├───playbooks
│   ├───site.yml                       # 🚀 Main orchestration
│   ├───validation/                    # ✅ Pre-flight and validation checks
│   │   ├───preflight_checks.yml      # 🚀 Environment readiness validation
│   │   └───test_vcenter.yml          # 🧪 vCenter connectivity testing
│   └───provisioning
│       ├───join_domain                # 🔧 Domain joining operations
│       │   ├───linux
│       │   │       join_linux_vm.yml
│       │   └───windows
│       │           join_windows_vm.yml
│       └───testing
│               domain_validation.yml  # ✅ Domain join validation
│
└───roles
    └───provisioning
        ├───join_domain                # 🔧 Domain joining logic
        │   ├───defaults/main.yml
        │   ├───meta/main.yml
        │   └───tasks/
        │           linux.yml
        │           main.yml
        │           windows.yml
        └───join_validation            # ✅ Validation logic
            ├───defaults/main.yml
            ├───meta/main.yml
            └───tasks/
                    main.yml               # 🎯 OS detection and validation orchestration
                    linux_validation.yml   # 🐧 Linux domain validation (realm list, SSSD)
                    windows_validation.yml # 🪟 Windows domain validation (native modules)
```

## 🎯 **Features**

- **🚀 GitHub Workflow-Driven**: Fully automated CI/CD pipeline for domain operations
- **🏗️ Clean Architecture**: Organized playbook and role structure
- **⚙️ Single Configuration**: All values centralized in `configs/project_config.yml`
- **🐧🪟 OS-Specific Logic**: Comprehensive Linux (`realm list`, SSSD) and Windows (native modules) validation
- **🔧 Role-Based Design**: Modular and reusable components
- **🐳 Containerized Execution**: Secure, isolated Ansible execution environment
- **🎯 Dynamic Host Targeting**: Automatically targets VMs based on configuration
- **📋 Production-Grade Validation**: Industry-standard domain membership verification
- **🔒 Secure Credential Management**: GitHub Secrets integration
- **🌐 Air-Gapped Ready**: No external dependencies or internet connectivity required

## 🚀 **Quick Start**

### 1. **Configure Project Settings**

Edit `configs/project_config.yml` - this is your **single source of truth**

```yaml
# 1. Domain Information
domain:
  name: "example.com"                    # Your domain name
  admin_user: "{{ secrets.DOMAIN_ADMIN_USER }}"       # Set in GitHub Secrets
  admin_password: "{{ secrets.DOMAIN_ADMIN_PASSWORD }}" # Set in GitHub Secrets
  linux_ou: "OU=Linux,OU=Servers,DC=example,DC=com"  # Linux OU path
  windows_ou: "OU=Windows,OU=Servers,DC=example,DC=com" # Windows OU path

# 2. Target VM Information
target_vm:
  name: "ansible-test-machine"                         # VM name
  ip_address: "10.x.x.x"                     # VM IP address
  os_type: "linux"                           # "linux" or "windows"

# 3. Network Settings
network:
  dns_server: "10.x.x.x"                    # Domain DNS server
  domain_controller: "dc01.example.com"              # Domain controller
```

### 2. **Set GitHub Secrets**

Configure the following secrets in your GitHub repository:

**Required Secrets:**

- `ANSIBLE_USER`: SSH/WinRM username for VM access
- `ANSIBLE_SSH_PASS`: SSH/WinRM password for VM access
- `ANSIBLE_BECOME_PASS`: Sudo/root password for privilege escalation
- `DOMAIN_ADMIN_USER`: Domain administrator username
- `DOMAIN_ADMIN_PASSWORD`: Domain administrator password

**Optional Secrets (for vCenter integration):**

- `VCENTER_HOSTNAME`: vCenter server hostname
- `VCENTER_USERNAME`: vCenter username
- `VCENTER_PASSWORD`: vCenter password
- `VCENTER_PORT`: vCenter port (default: 443)
- `VCENTER_VALIDATE_CERTS`: Certificate validation (default: false)

### 3. **Add Your Servers**

The inventory is **automatically generated** from your `project_config.yml`. The template at `inventory/hosts.ini.template` creates:

```ini
# Generated inventory example
[domain_linux]
ansible-test-machine ansible_host=10.x.x.x ansible_user=root

[domain_windows]
# Windows VMs would appear here based on os_type in config
```

**Dynamic Host Targeting:** The validation playbook automatically targets:

```yaml
hosts: "{{ (lookup('file', '/workspace/configs/project_config.yml') | from_yaml).target_vm.name }}-join"
```

### 4. **Create Pull Request**

1. Make your configuration changes
2. Create a pull request to the `main` branch
3. The workflow will automatically:
   - Test vCenter connectivity
   - Run pre-flight validation
   - Execute domain join automation
   - Provide detailed results in PR comments

### 5. **Manual Workflow Trigger**

You can also manually trigger the workflow:

1. Go to **Actions** tab in GitHub
2. Select **Domain Join Automation**
3. Click **Run workflow**
4. Choose your branch and click **Run workflow**

## 🔧 **Configuration**

### **Single Configuration File (`configs/project_config.yml`)**

This file contains **EVERYTHING**:

- **Domain settings** and credentials
- **Target group definitions**
- **Network configuration**
- **Validation parameters**
- **Security settings**
- **Performance tuning**
- **File paths**
- **Ansible configuration**
- **GitHub workflow settings**
- **vCenter integration settings**

### **Static Inventory (`inventory/hosts.ini`)**

- **Manual server management** (no automatic discovery)
- Server group definitions
- Host-specific variables
- Network connectivity settings

## 🎭 **Roles**

### **`provisioning.join_domain`**

- **`linux.yml`**: Linux domain joining tasks
- **`windows.yml`**: Windows domain joining tasks
- **`main.yml`**: Role entry point and OS detection

### **`provisioning.join_validation`**

- **`main.yml`**: OS detection and validation orchestration
- **`linux_validation.yml`**: Comprehensive Linux domain validation
   - `realm list` validation with detailed parsing
   - SSSD service status and configuration checks
   - Multiple authentication method testing
   - DNS resolution and connectivity verification
- **`windows_validation.yml`**: Comprehensive Windows domain validation
   - Native Windows domain membership modules
   - Domain controller connectivity testing
   - Group Policy and RSAT feature validation
   - Trust relationship and computer account verification

## 📋 **Playbooks**

### **`site.yml`**

- Main orchestration playbook
- Loads centralized project configuration
- Executes complete automation workflow

### **`validation/preflight_checks.yml`**

- **Pre-flight environment validation**
- Checks configuration completeness
- Validates network connectivity
- Verifies file structure and roles
- Ensures environment readiness

### **`provisioning/join_domain/linux/join_linux_vm.yml`**

- Linux VM domain join execution
- Includes domain join role with Linux tasks

### **`provisioning/join_domain/windows/join_windows_vm.yml`**

- Windows VM domain join execution
- Includes domain join role with Windows tasks

### **`provisioning/testing/domain_validation.yml`**

- **Dynamic host targeting** from `project_config.yml`
- **Comprehensive domain validation** for both Linux and Windows
- **Detailed status reporting** with troubleshooting information
- **Production-grade validation** covering:
   - Domain membership verification
   - Service status and configuration
   - Authentication and connectivity testing
   - DNS resolution and trust relationships

## 🚀 **Usage Examples**

### **GitHub Workflow Automation**

The workflow automatically runs when you:

1. **Push to main branch** (including PR merges)
2. **Manually trigger** via GitHub Actions UI
3. **Workflow dispatch** with custom parameters

### **Workflow Steps**

The automation includes these steps:

1. **Configuration Parsing** - Extracts settings from `configs/project_config.yml`
2. **Dynamic Inventory Generation** - Creates `vm_config.yml` and resolved inventory
3. **SSH Connectivity Test** - Verifies VM accessibility
4. **Domain Validation Execution** - Runs comprehensive validation checks:
   - **Linux**: `realm list`, SSSD status, authentication testing
   - **Windows**: Domain membership, trust relationships, Group Policy
5. **Detailed Reporting** - Provides comprehensive status and troubleshooting output

### **Manual Workflow Execution**

You can manually trigger the workflow:

1. Go to **Actions** → **Domain Join Automation**
2. Click **Run workflow**
3. Select branch and click **Run workflow**

### **Workflow Output**

The workflow provides:

- **Real-time logs** in GitHub Actions
- **PR comments** with execution results
- **Success/failure status** for each step
- **Detailed error messages** for troubleshooting

## 🔒 **Security**

- **GitHub Secrets**: Sensitive credentials stored securely in GitHub
- **Containerized Execution**: Isolated Ansible execution environment
- **Password Management**: Secure credential handling through secrets
- **Access Control**: Role-based permissions and validation
- **Air-Gapped**: No external network dependencies
- **PR-Based Changes**: All configuration changes through Git pull requests

## 🧪 **Testing**

### **GitHub Workflow Testing**

The workflow automatically runs comprehensive tests:

1. **Configuration Validation** - Validates YAML syntax and structure
2. **vCenter Connectivity** - Tests vCenter server connectivity
3. **Pre-flight Checks** - Validates environment readiness
4. **Domain Join Execution** - Runs actual domain joining
5. **Post-join Validation** - Verifies successful domain membership

### **Manual Testing (Local Development)**

For local development and testing:

```bash
# Syntax check
ansible-playbook --syntax-check playbooks/provisioning/testing/domain_validation.yml

# Dry run
ansible-playbook --check playbooks/provisioning/testing/domain_validation.yml

# Domain validation (main use case)
ansible-playbook -i inventory/hosts_resolved.ini playbooks/provisioning/testing/domain_validation.yml --tags validation

# Pre-flight validation
ansible-playbook -i inventory/hosts.ini playbooks/validation/preflight_checks.yml
```

### **Container Testing**

Test using the containerized environment (matches GitHub Actions exactly):

```bash
# Use Podman directly (Linux/WSL/Windows)
podman run --rm --privileged \
  -v "$(pwd):/workspace:Z" \
  -e ANSIBLE_HOST_KEY_CHECKING=False \
  --network host \
  -w /workspace \
  docker.io/vzamora/join-operations:latest \
  ansible-playbook -i inventory/hosts_resolved.ini \
  playbooks/provisioning/testing/domain_validation.yml \
  --tags validation
```

## 🔧 **Customization**

### **Adding New OS Support**

1. Create new task file in `roles/provisioning/join_domain/tasks/`
2. Update `roles/provisioning/join_domain/tasks/main.yml`
3. Add OS-specific variables to `configs/project_config.yml`

### **Extending Validation**

1. Add new validation tasks to `roles/provisioning/join_validation/tasks/main.yml`
2. Update validation parameters in `configs/project_config.yml`

### **Adding Pre-flight Checks**

1. Add new validation tasks to `playbooks/validation/preflight_checks.yml`
2. Update configuration validation in the preflight playbook

### **Managing Target VMs**

1. **Change target VM**: Update `target_vm` section in `configs/project_config.yml`
2. **Switch domains**: Modify `domain.name` and related settings
3. **Update network settings**: Change `network.dns_server` and `network.domain_controller`
4. **OS type switching**: Set `target_vm.os_type` to `linux` or `windows`

**Note**: The inventory is automatically generated from your configuration - no manual inventory editing needed!

### **Modifying Project Settings**

**Everything is in one place**: `configs/project_config.yml`

- Change domain settings
- Modify target groups
- Adjust validation parameters
- Update network configuration
- Tune performance settings

## 🐛 **Troubleshooting**

### **Common Issues**

- **Authentication Failed**: Check GitHub secrets (`DOMAIN_ADMIN_USER`, `DOMAIN_ADMIN_PASSWORD`)
- **Connection Refused**: Verify network connectivity and SSH/WinRM access to VM
- **Host Not Found**: Check `target_vm.name` and `target_vm.ip_address` in `project_config.yml`
- **Domain Validation Failures**:
   - **Linux**: Check `realm list` output and SSSD service status
   - **Windows**: Verify domain membership and trust relationships
- **Workflow Failures**: Check GitHub Actions logs for detailed error messages

### **GitHub Workflow Troubleshooting**

1. **Check Workflow Logs**: Go to Actions tab and view detailed logs
2. **Verify Secrets**: Ensure all required GitHub secrets are configured
3. **Check Configuration**: Validate `configs/project_config.yml` syntax
4. **Review PR Comments**: Workflow results are posted in PR comments

### **Domain Validation Troubleshooting**

For detailed validation troubleshooting:

```bash
# Run validation with maximum verbosity
ansible-playbook -i inventory/hosts_resolved.ini \
  playbooks/provisioning/testing/domain_validation.yml \
  --tags validation -vvv

# Check specific validation components
# Linux: realm list, SSSD status, DNS resolution
# Windows: Domain membership, trust relationships, Group Policy
```

### **Debug Mode**

For local debugging:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/site.yml -vvv
```

### **Container Debugging**

Debug using the exact containerized environment from GitHub Actions:

```bash
# Interactive container debugging
podman run -it --rm --privileged \
  -v "$(pwd):/workspace:Z" \
  -e ANSIBLE_HOST_KEY_CHECKING=False \
  --network host \
  -w /workspace \
  docker.io/vzamora/join-operations:latest /bin/bash

# Then inside container:
ansible-playbook -i inventory/hosts_resolved.ini \
  playbooks/provisioning/testing/domain_validation.yml \
  --tags validation -vvv
```

## 📚 **Documentation**

- **Ansible Documentation**: [docs.ansible.com](https://docs.ansible.com)
- **GitHub Actions**: [GitHub Actions Documentation](https://docs.github.com/en/actions)
- **Role Development**: [Ansible Role Guidelines](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)
- **Docker Documentation**: [Docker Documentation](https://docs.docker.com/)

## 🤝 **Contributing**

1. Follow the established project structure
2. Use role-based design for new functionality
3. **Update `configs/project_config.yml` for new variables**
4. Maintain clean and organized code structure
5. Keep air-gapped compatibility in mind
6. **Use GitHub pull requests** for all changes
7. **Test changes** using the GitHub workflow before merging

## 📄 **License**

This project follows the same licensing terms as your organization's standard practices.

## 🌐 **Air-Gapped Environment Notes**

### **What Works Offline:**

- ✅ **Dynamic inventory generation** from configuration
- ✅ **Domain validation automation** (Linux and Windows)
- ✅ **Role-based architecture** with comprehensive validation
- ✅ **GitHub workflow automation** (requires GitHub connectivity)
- ✅ **Containerized execution** with production-grade validation
- ✅ **Single configuration file** (`project_config.yml`)
- ✅ **Comprehensive domain validation** with detailed reporting
- ✅ **Dynamic host targeting** based on configuration

### **What Requires Internet:**

- ❌ **GitHub Actions execution** (requires GitHub connectivity)
- ❌ **Container image pulls** (pre-pull `docker.io/vzamora/join-operations:latest`)
- ❌ **External collections** (all required collections are pre-installed in container)
- ❌ **vCenter API calls** (optional, only if using vCenter integration)

### **Best Practices for Air-Gapped:**

1. **Pre-pull container images** before air-gapped deployment
2. **Use single configuration file** (`project_config.yml`) for all settings
3. **Dynamic inventory generation** eliminates manual inventory management
4. **Self-contained validation** with comprehensive domain checks
5. **Production-grade validation** covers all critical domain components
6. **Containerized execution** ensures consistent environments
7. **Comprehensive logging** for troubleshooting without external dependencies
8. **GitHub workflow** for automated validation (when GitHub connectivity available)
