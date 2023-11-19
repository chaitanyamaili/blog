---
layout: default
parent: Post
title: Terraform - import
nav_order: 2
---

## Import

Terraform can import existing infrastructure resources. This functionality lets you bring existing resources under Terraform management

### Command

`terraform [global options] import [options] ADDR ID`

```
ADDR specified is the address to import the resource to. 
ID is a resource-specific ID to identify that resource being imported.
```

### When should we use

Terraform import can be used in two scenario:
 - Terraforming the existing infrastructure
 - Restructing the infrastructure code

For the "Restructing the infrastructure code" we need to first clean the previous terraform state and then use the terraform import comands to import as per the updated strcuture.

`terraform state rm <resource-address>`

`terraform import <new-resource-address> <resource-id>`