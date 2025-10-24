# Cleaning Up

In this lab you will delete the compute resources created during this tutorial.

## Compute Instances

Previous versions of this guide made use of GCP resources for various aspects of compute and networking. The current version is agnostic, and all configuration is performed on the `jumpbox`, `server`, or nodes.

> **Clean up importance**: Deleting unused resources prevents unnecessary costs and security risks. In cloud environments, always remove test clusters when done. For production clusters, implement proper lifecycle management and backup strategies before decommissioning.

Clean up is as simple as deleting all virtual machines you created for this exercise.

Next: [Start Over](../README.md)
