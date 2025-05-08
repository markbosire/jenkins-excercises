# Jenkins Project-based Matrix Authorization Project

## Overview

This project demonstrates how to implement **Project-based Matrix Authorization** in Jenkins, which provides granular permission control at the individual project/job level while maintaining global security settings. This approach allows different permission sets for different projects within the same Jenkins instance.

## Key Features

- Project-specific permission matrices
- Inherited permissions from global matrix
- Role-based access control at project level
- Sample job configurations with different permission schemes
- Documentation on combining global and project-level permissions

## Configuration Steps

### 1. Set Up Global Matrix Authorization (Base Layer)

1. Go to **Manage Jenkins** > **Configure Global Security**
2. Under Authorization, select **Matrix-based security**
3. Configure base permissions that will apply to all projects
4. Add admin users/groups with full permissions

### 2. Enable Project-based Matrix Authorization

1. In the same security configuration page:
2. Check **Project-based Matrix Authorization** option
3. Save the configuration

### 3. Configure Project-specific Permissions

For each project/job:

1. Open the project configuration
2. Locate the **Enable project-based security** checkbox (under General or Security section)
3. Configure the permission matrix specific to this project
4. The project matrix will combine with the global matrix (most restrictive applies)

## Example Project Configurations

### Project A (Strict Access)
```
Permissions       | Admin | Lead Dev | QA
-----------------------------------------
Job/Build        | ✓     | ✓        |
Job/Configure    | ✓     |          |
Job/Read         | ✓     | ✓        | ✓
```

### Project B (Open Access)
```
Permissions       | Admin | Dev Team | Contractor
-----------------------------------------------
Job/Build        | ✓     | ✓        | ✓
Job/Read         | ✓     | ✓        | ✓
```

## Best Practices

1. **Global Matrix**: Set baseline permissions that apply to all projects
2. **Project Matrix**: Only override what needs to be different
3. **Inheritance**: Remember permissions are cumulative (user needs permission in both matrices)
4. **Groups**: Use groups rather than individual users
5. **Documentation**: Maintain a permission matrix document

## Testing Your Setup

1. Create test users for each role
2. Verify access to different projects
3. Check permission inheritance:
   - A user needs permission in both global and project matrices
   - Denials in either matrix will prevent access

## Troubleshooting

- **"Missing permission" errors**: Check both global and project matrices
- **Unexpected access**: Verify there are no conflicts between matrices
- **Inheritance issues**: Remember the AND relationship between matrices

## Security Considerations

1. Always have at least one admin user configured in the global matrix
2. Audit project permissions regularly
3. Consider using the **Role Strategy Plugin** for more complex scenarios
4. Document any permission overrides at the project level
