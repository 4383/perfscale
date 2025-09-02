# Rally Performance Testing Implementation Specification

**Prompt for AI Assistant**: You are tasked with implementing Rally-based performance and scale testing for an OpenStack project repository. This specification provides a complete template following the approach established in the Neutron project and successfully applied to Nova.

## Task Overview

Analyze the current OpenStack repository and implement a complete Rally performance testing infrastructure. The implementation should:

1. **Detect the OpenStack service type** from the repository structure and configuration files
2. **Create the standard Rally directory structure** with appropriate documentation
3. **Generate service-specific performance test scenarios** based on the service's core operations
4. **Include proper CI integration structure** for automated testing
5. **Follow OpenStack community standards** and best practices

## Step 1: Service Detection and Analysis

Before implementing Rally testing, analyze the repository to determine the OpenStack service:

### Automatic Service Detection
1. **Check setup.cfg or setup.py** for project name and description
2. **Examine directory structure** for service-specific modules (nova/, cinder/, neutron/, etc.)
3. **Look for service entry points** in configuration files
4. **Identify core resource types** the service manages

### Service Mapping
- **Nova (Compute)**: Manages servers, flavors, keypairs, images
- **Cinder (Block Storage)**: Manages volumes, volume types, snapshots
- **Neutron (Networking)**: Manages networks, subnets, ports, routers
- **Ironic (Bare Metal)**: Manages nodes, chassis, ports
- **Swift (Object Storage)**: Manages containers, objects
- **Heat (Orchestration)**: Manages stacks, templates, resources
- **Keystone (Identity)**: Manages users, projects, tokens, policies
- **Glance (Image)**: Manages images, image members
- **Trove (Database)**: Manages database instances, backups
- **Zaqar (Messaging)**: Manages queues, messages

## Step 2: Directory Structure Creation

Create the following directory structure in the project root:

```
rally-jobs/
├── README.rst
├── task-<project>.yaml
├── plugins/
│   ├── __init__.py
│   └── README.rst
└── extra/
    └── README.rst
```

### Implementation Commands
```bash
mkdir -p rally-jobs/plugins rally-jobs/extra
touch rally-jobs/plugins/__init__.py
```

## Step 3: Documentation Files

### Main README.rst

Document your Rally setup with links to relevant resources:

```rst
Rally job related files
=======================

This directory contains rally tasks and plugins that are run by OpenStack CI.

Structure
---------

* plugins - directory where you can add rally plugins. Almost everything in
  Rally is a plugin. Benchmark context, Benchmark scenario, SLA checks, Generic
  cleanup resources, ....

* extra - all files from this directory will be copy pasted to gates, so you
  are able to use absolute paths in rally tasks.
  Files will be located in ~/.rally/extra/*

* task-<project>.yaml is a task that is run in gates against OpenStack with
  <Project> Service deployed by DevStack

Useful links
------------

* More about Rally: https://rally.readthedocs.io/en/latest/
* Rally release notes: https://rally.readthedocs.io/en/latest/project_info/release_notes/archive.html
* How to add rally-gates: http://rally.readthedocs.io/en/latest/quick_start/gates.html
* About plugins: https://rally.readthedocs.io/en/latest/plugins/index.html
* Rally samples: https://github.com/openstack/rally/tree/master/samples/
```

## Step 4: Task Configuration File

Create `task-{service_name}.yaml` with service-specific scenarios. Replace `{service_name}` with the detected service name (nova, cinder, neutron, etc.).

```yaml
{% set image_name = "^(cirros.*-disk|TestVM)$" %}
{% set flavor_name = "m1.tiny" %}

---
version: 2
title: Rally Task for OpenStack <Project> CI
description: >
  The task contains various scenarios to test <Project> performance
subtasks:
  -
    title: <Resource> related workloads
    workloads:
      -
        description: Check performance of list_<resources> action
        scenario:
          <Project><Resource>.create_and_list_<resources>: {}
        runner:
          constant:
            times: 50
            concurrency: 10
        contexts:
          users:
            tenants: 1
            users_per_tenant: 1
          quotas:
            <project>:
              <resource>: -1
        sla:
          max_avg_duration_per_atomic:
            <project>.list_<resources>: 15
          failure_rate:
            max: 0
```

## Step 5: Service-Specific Scenario Selection

Based on the detected service, implement the appropriate scenarios:

### Nova (Compute Service)
**Core Operations**: Server lifecycle, keypair management, flavor operations
**Scenarios to implement**:
- `NovaServers.boot_and_delete_server` - Primary server lifecycle test
- `NovaServers.boot_and_list_server` - Server creation and listing
- `NovaServers.snapshot_server` - Image creation from servers
- `NovaKeypair.create_and_delete_keypair` - Keypair management
- `NovaKeypair.create_and_list_keypairs` - Keypair listing
- `NovaFlavors.list_flavors` - Flavor enumeration

### Cinder (Block Storage Service)
**Core Operations**: Volume lifecycle, attachment operations
**Scenarios to implement**:
- `CinderVolumes.create_and_delete_volume` - Primary volume lifecycle
- `CinderVolumes.create_and_attach_volume` - Volume attachment
- `CinderVolumes.create_and_list_volume` - Volume listing
- `CinderVolumeTypes.create_and_delete_volume_type` - Volume type management
- `CinderVolumeBackups.create_and_delete_volume_backup` - Backup operations

### Neutron (Networking Service)
**Core Operations**: Network topology management
**Scenarios to implement**:
- `NeutronNetworks.create_and_delete_networks` - Network lifecycle
- `NeutronNetworks.create_and_list_networks` - Network listing
- `NeutronNetworks.create_and_delete_subnets` - Subnet management
- `NeutronNetworks.create_and_delete_ports` - Port management
- `NeutronNetworks.create_and_delete_routers` - Router operations

### Ironic (Bare Metal Service)
**Core Operations**: Node provisioning and management
**Scenarios to implement**:
- `IronicNodes.create_and_delete_node` - Node lifecycle
- `IronicNodes.create_and_list_node` - Node listing
- Custom scenarios for power state management
- Custom scenarios for provisioning workflows

### Swift (Object Storage Service)
**Core Operations**: Container and object management
**Scenarios to implement**:
- `SwiftObjects.create_container_and_object_then_list_objects` - Object operations
- `SwiftObjects.create_container_and_object_then_delete_all` - Cleanup operations
- `SwiftObjects.list_objects_in_containers` - Listing operations

### Heat (Orchestration Service)
**Core Operations**: Stack lifecycle management
**Scenarios to implement**:
- `HeatStacks.create_and_delete_stack` - Stack lifecycle
- `HeatStacks.create_and_list_stack` - Stack listing
- `HeatStacks.create_update_delete_stack` - Stack updates

### Keystone (Identity Service)
**Core Operations**: User, project, and token management
**Scenarios to implement**:
- `KeystoneBasic.create_and_delete_user` - User management
- `KeystoneBasic.create_and_delete_project` - Project lifecycle
- `KeystoneBasic.get_entities` - Entity enumeration
- `KeystoneBasic.create_and_list_users` - User listing

### Glance (Image Service)
**Core Operations**: Image lifecycle management
**Scenarios to implement**:
- `GlanceImages.create_and_delete_image` - Image lifecycle
- `GlanceImages.create_and_list_image` - Image listing
- `GlanceImages.list_images` - Image enumeration

### Additional Service Types
For other services (Trove, Zaqar, etc.), follow the pattern of identifying core CRUD operations and implementing corresponding Rally scenarios.

### 5. Custom Plugins Development

Create service-specific plugins in `plugins/` directory:

```python
# plugins/custom_scenarios.py
from rally_openstack import consts
from rally_openstack.task.scenarios.<project> import utils
from rally.task import scenario

@scenario.configure(name="<Project><Resource>.custom_scenario", 
                   platform="openstack")
class CustomScenario(utils.<Project>ScenarioMixin, scenario.Scenario):
    """Custom scenario for <Project> service."""
    
    def run(self, **kwargs):
        """Run the custom scenario."""
        # Implementation here
        pass
```

### 6. Extra Files Configuration

Place any additional files needed by scenarios in `extra/`:

```rst
Extra files
===========

All files from this directory will be copy pasted to gates, so you are able to
use absolute path in rally tasks. Files will be in ~/.rally/extra/*
```

### 7. CI Integration

#### Zuul Configuration
Add Rally job to your `.zuul.yaml`:

```yaml
- job:
    name: <project>-rally
    parent: rally-task-<project>
    timeout: 7800
    vars:
      rally_task: rally-jobs/task-<project>.yaml
      devstack_plugins:
        rally-openstack: https://opendev.org/openstack/rally-openstack
      devstack_services:
        rally: true
    required-projects:
      - openstack/rally-openstack

- project:
    check:
      jobs:
        - <project>-rally
    gate:
      jobs:
        - <project>-rally
```

#### DevStack Configuration
Ensure your service is properly configured in DevStack for Rally testing.

### 8. Performance Tuning Guidelines

#### Runner Configuration
- **times**: Number of iterations to run
- **concurrency**: Number of parallel workers
- Start with low concurrency (2-5) and increase gradually

#### Context Configuration
- **users**: Configure appropriate tenant/user ratios
- **quotas**: Set to -1 (unlimited) for most resources during testing
- **network**: Include if your service requires networking

#### SLA Configuration
- **max_avg_duration_per_atomic**: Set realistic time limits for operations
- **failure_rate**: Usually set to 0 for CI, but may allow some failures for unstable scenarios

### 9. Common Scenarios by Service Type

#### Database Services (Trove, etc.)
- Instance creation/deletion
- Database operations
- Backup/restore operations

#### Messaging Services (Zaqar, etc.)
- Queue creation/deletion
- Message posting/consuming
- Subscription management

#### Orchestration Services (Heat, etc.)
- Stack creation/deletion
- Template validation
- Resource updates

#### Identity Services (Keystone, etc.)
- User/project creation
- Token operations
- Policy enforcement

### 10. Best Practices

1. **Start Simple**: Begin with basic CRUD operations
2. **Gradual Complexity**: Add more complex scenarios over time
3. **Resource Cleanup**: Ensure all scenarios properly clean up resources
4. **Realistic Workloads**: Base scenarios on real-world usage patterns
5. **Monitor Resources**: Watch for resource leaks during development
6. **Version Control**: Keep Rally configurations in sync with service versions
7. **Documentation**: Document any custom scenarios and their purpose

### 11. Testing and Validation

#### Local Testing
```bash
# Install Rally
pip install rally-openstack

# (optional) May require intermediate steps like
rally db create # See neutron's rally setup

# Run specific scenario
rally task start rally-jobs/task-<project>.yaml

# Generate reports
rally task report --out report.html
```

#### CI Integration Testing
1. Test in your development environment first
2. Validate scenarios don't cause resource exhaustion
3. Ensure proper cleanup in failure cases
4. Monitor CI job execution times

### 12. Troubleshooting

#### Common Issues
- **Quota Exceeded**: Increase quotas or reduce concurrency
- **Timeouts**: Adjust SLA limits or improve service performance
- **Resource Conflicts**: Ensure proper resource isolation
- **Authentication Failures**: Verify service configuration

#### Debugging
- Enable Rally debug logging
- Check service logs for errors
- Validate DevStack configuration
- Review scenario implementation

### 13. Maintenance and Updates

- **Regular Review**: Update scenarios as service capabilities evolve
- **Performance Baselines**: Track performance trends over time
- **Scenario Cleanup**: Remove obsolete or ineffective scenarios
- **Plugin Updates**: Keep custom plugins compatible with Rally updates

## Implementation Checklist

Execute these steps in order to implement Rally performance testing:

### Phase 1: Analysis and Setup
- [ ] Detect OpenStack service type from repository structure
- [ ] Identify service name and core resources managed
- [ ] Create rally-jobs directory structure
- [ ] Create plugins/__init__.py with proper license header

### Phase 2: Documentation
- [ ] Create main rally-jobs/README.rst with service-specific content
- [ ] Create plugins/README.rst with plugin development guidance
- [ ] Create extra/README.rst for additional files

### Phase 3: Test Configuration
- [ ] Create task-{service}.yaml with appropriate scenarios
- [ ] Configure service-specific workloads and scenarios
- [ ] Set appropriate concurrency, timing, and SLA parameters
- [ ] Include necessary contexts (users, quotas, networking if needed)

### Phase 4: Validation and Testing
- [ ] Validate YAML syntax and Rally scenario names
- [ ] Test scenarios locally if Rally environment available
- [ ] Verify resource cleanup after scenario execution
- [ ] Document any custom scenarios implemented

### Phase 5: Commit and Integration
- [ ] Commit changes with descriptive message referencing https://github.com/4383/perfscale
- [ ] Consider CI integration following provided Zuul configuration examples
- [ ] Set up monitoring for performance regression detection

## Expected Deliverables

Upon completion, the implementation should provide:

1. **Complete rally-jobs directory structure** with all required files
2. **Service-specific performance test scenarios** covering core operations
3. **Comprehensive documentation** for maintenance and extension
4. **CI-ready configuration** following OpenStack standards
5. **Commit message** referencing the perfscale template source

## Success Criteria

- [ ] All Rally scenarios execute without syntax errors
- [ ] Test scenarios cover the service's primary use cases
- [ ] Resource cleanup is properly handled
- [ ] Documentation is complete and follows RST format
- [ ] Implementation follows OpenStack community patterns

## Resources and References

- [Rally Documentation](https://rally.readthedocs.io/)
- [Rally OpenStack Plugin](https://github.com/openstack/rally-openstack)
- [Neutron Rally Implementation](https://opendev.org/openstack/neutron/src/branch/master/rally-jobs)
- [OpenStack Performance Testing](https://docs.openstack.org/performance-docs/)
- [Perfscale Template Source](https://github.com/4383/perfscale)

---

**Note**: This specification was derived from successful implementation in the Nova project and provides a complete template for standardizing Rally performance testing across all OpenStack services.
