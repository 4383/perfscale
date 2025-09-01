# Performance and Scale Testing Setup Guide for OpenStack Projects

This document provides detailed guidance on setting up Rally-based performance and scale testing for OpenStack projects, following the approach established in the Neutron project.

## Overview

Rally is a benchmarking framework that provides automated performance testing for OpenStack services. This guide helps you implement a similar performance testing infrastructure as used in Neutron for other OpenStack deliverables like Nova, Ironic, Cinder, etc.

## Directory Structure

Create the following directory structure in your project root:

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

## Step-by-Step Implementation

### 1. Create the Rally Jobs Directory

```bash
mkdir -p rally-jobs/plugins rally-jobs/extra
touch rally-jobs/plugins/__init__.py
```

### 2. Create README.rst

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

### 3. Create Task Configuration File

Create `task-<project>.yaml` based on your service's key operations. Here's a template structure:

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

### 4. Service-Specific Scenarios

#### For Nova (Compute)
Focus on server lifecycle operations:
- `NovaServers.boot_and_delete_server`
- `NovaServers.boot_and_list_server`
- `NovaServers.snapshot_server`
- `NovaServers.resize_server`
- `NovaKeypair.create_and_delete_keypair`

#### For Cinder (Block Storage)
Focus on volume operations:
- `CinderVolumes.create_and_delete_volume`
- `CinderVolumes.create_and_attach_volume`
- `CinderVolumes.create_and_list_volume`
- `CinderVolumeTypes.create_and_delete_volume_type`

#### For Ironic (Bare Metal)
Focus on node management:
- `IronicNodes.create_and_delete_node`
- `IronicNodes.create_and_list_node`
- Custom scenarios for provisioning workflows

#### For Swift (Object Storage)
Focus on object operations:
- `SwiftObjects.create_container_and_object_then_list_objects`
- `SwiftObjects.create_container_and_object_then_delete_all`

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

## Example Implementation Checklist

- [ ] Create rally-jobs directory structure
- [ ] Write service-specific task file
- [ ] Implement custom scenarios if needed
- [ ] Configure CI integration
- [ ] Test locally
- [ ] Validate in CI environment
- [ ] Document custom scenarios
- [ ] Set performance baselines
- [ ] Monitor and maintain

## Resources and References

- [Rally Documentation](https://rally.readthedocs.io/)
- [Rally OpenStack Plugin](https://github.com/openstack/rally-openstack)
- [Neutron Rally Implementation](https://opendev.org/openstack/neutron/src/branch/master/rally-jobs)
- [OpenStack Performance Testing](https://docs.openstack.org/performance-docs/)

This approach ensures consistent performance testing across OpenStack projects while allowing for service-specific customizations and scenarios.
