#terraform #cloudguard

This resource provides the Target resource in Oracle Cloud Infrastructure Cloud Guard service.
## Example Usage

```hcl
resource "oci_cloud_guard_target" "example_target" {
  compartment_id = var.compartment_ocid
  display_name   = "MyCloudGuardTarget"
  target_resource_id = var.target_resource_ocid
  target_resource_type = "COMPARTMENT"
  description = "Target for security monitoring"
}
```