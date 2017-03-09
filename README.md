# AWS Customer Gateway
======================

This module is intended to deploy a single AWS Customer Gateway and other resources necessary to configure a functioning VPN Connection.

Module Input Variables
----------------------

- `name`   - Unique name used to label the VPN Gateway and Customer Gateway.
- `vpn_gateway_id` - VPN Gateway to associate with Customer Gateway and VPN Connection.
- `ip_address` - The IP address of the gateway's Internet-routable external interface.
- `bgp_asn` - BGP Autonomous System Number. If BGP is not in use, then by convention set this value to 65000.
- `destination_cidr_blocks` - A comma separated list of CIDR blocks which sit behind the Customer Gateway device and should be routed over the VPN connection.
- `route_table_ids` - (optional) A comma separated list of route tables ids. This must be provided if you plan to create static routes for the destination_cidr_blocks in each route table.
- `route_table_count` - (optional) The total number of tables in the route_table_ids list. This must be provided if route_table_ids is set. This is necessary since value of `count` cannot be computed in modules.
- `static_routes_only` - Determines whether routes learned from BGP will be propagated to route tables. If set to true, you must have vgw route propagation enabled on route tables, and of course you must be running BGP. Accepts true or false.
- `add_static_routes_to_tables` - Determines whether static routes will be added to all route tables in route_table_ids list or if vgw route propagation will be used instead. If set to true, then route_table_ids, route_table_count, and destination_cidr_blocks must also be provided.

Usage 
-----
```js
module "vpc" {
  source = "github.com/terraform-community-modules/tf_aws_vpc_only"

  #properties omitted
}

module "public_subnet" {
  source = "github.com/terraform-community-modules/tf_aws_public_subnet"

  #properties omitted
}

module "private_subnet" {
  source = "github.com/terraform-community-modules/tf_aws_private_subnet_nat_gateway"

  #properties omitted
}

module "vpn" {
  source = "github.com/terraform-community-modules/tf_aws_vpn_gw"

  name   = "tf-aws-vpn-example"
  vpc_id = "${module.vpc.vpc_id}"
}


module "stockholm_cgw" {
  source = "github.com/terraform-community-modules/tf_aws_customer_gw"

  name = "stockholm"

  vpn_gateway_id     = "${module.vpn.vgw_id}"
  ip_address         = "66.66.66.66"
  bgp_asn            = "65000"
  static_routes_only = true

  add_static_routes_to_tables  = true
  route_table_ids              = "${concat(module.public_subnet.public_route_table_ids, module.private_subnet.private_route_table_ids)}"
  route_table_count            = 6
  destination_cidr_blocks      = ["10.1.1.0/24", "10.100.1.0/24"]
}



```

Outputs
-------
- `cgw_id` - ARN for the created Customer Gateway.
- `cgw_ip_address` - The IP address of the gateway's Internet-routable external interface.
- `cgw_bgp_asn` - The gateway's Border Gateway Protocol (BGP) Autonomous System Number (ASN).

Author
------
Created and maintained by [Shayne Clausson](https://github.com/sclausson)