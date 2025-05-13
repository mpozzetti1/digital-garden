#terraform #cloudguard 

This resource provides the Detector Recipe resource in Oracle Cloud Infrastructure Cloud Guard service.
## Example Usage

```hcl
resource "oci_cloud_guard_detector_recipe" "business-activity-recipe" {
  compartment_id            = var.business
  display_name              = "Business Activity Recipe"
  description               = "Detects suspicious IAM changes"
  detector                  = "IAAS_ACTIVITY_DETECTOR"
  source_detector_recipe_id = var.recipe-activity

  detector_rules {
    detector_rule_id = "IAM_ADMIN_ACTIVITY"
    details {
      is_enabled = true
      risk_level = "HIGH"
    }
  }
}
```