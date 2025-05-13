#terraform #cloudguard

This resource provides the Cloud Guard Configuration resource in Oracle Cloud Infrastructure Cloud Guard service.
## Example Usage

```hcl
resource "oci_cloud_guard_cloud_guard_configuration" "test_cloud_guard_configuration" {
	#Required
	compartment_id = var.compartment_id
	reporting_region = var.cloud_guard_configuration_reporting_region
	status = var.cloud_guard_configuration_status

	#Optional
	self_manage_resources = var.cloud_guard_configuration_self_manage_resources
	service_configurations {
		#Required
		service_configuration_type = var.cloud_guard_configuration_service_configurations_service_configuration_type

		#Optional
		status = var.cloud_guard_configuration_service_configurations_status
	}
}
```