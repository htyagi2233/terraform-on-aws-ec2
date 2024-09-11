# Terraform For Loops, Lists, Maps and Count Meta-Argument

## Step-00: Pre-requisite Note
- We are using the `default vpc` in `us-east-1` region

## Step-01: Introduction
- Terraform Meta-Argument: `Count`
- **Terraform Lists & Maps**
  - List(string)
  - map(string)
- **Terraform for loops**
  - for loop with List
  - for loop with Map
  - for loop with Map Advanced
- **Splat Operators**
  - Legacy Splat Operator `.*.`
  - Generalized Splat Operator (latest)
  - Understand about Terraform Generic Splat Expression `[*]` when dealing with `count` Meta-Argument and multiple output values

## Step-02: c1-versions.tf 
- No changes

## Step-03: c2-variables.tf - Lists and Maps
```t
# AWS EC2 Instance Type - List (अगर हमें list में value देनी है और इस variable value को pick करेंगे तब हमें 1 और 2 देनी होगी यानि कौन सी value pick करनी है )
variable "instance_type_list" {
  description = "EC2 Instnace Type"
  type = list(string)
  default = ["t3.micro", "t3.small"]
}


# AWS EC2 Instance Type - Map (map में हम key-pair में value define करते है और variable value को pick करने के लिए हम only key को लेते है)
variable "instance_type_map" {
  description = "EC2 Instnace Type"
  type = map(string)
  default = {
    "dev" = "t3.micro"
    "qa"  = "t3.small"
    "prod" = "t3.large"
  }
}
```

## Step-04: c3-ec2securitygroups.tf and c4-ami-datasource.tf
- No changes to both files

## Step-05: c5-ec2instance.tf
```t
# How to reference List values ? (अगर हमें list में value दी है और इस variable value को pick करेंगे तब हमें 1 और 2 देनी होगी यानि कौन सी value pick करनी है )
instance_type = var.instance_type_list[1]

# How to reference Map values ? (map में हम key-pair में value define करते है और variable value को pick करने के लिए हम only key को लेते है)
instance_type = var.instance_type_map["prod"]

# Meta-Argument Count  (इसमें हम 2 resource create करने के लिए define कर रहे है ) 
count = 2

# count.index  (और दोनों resources में जब तब add hoga तब count-demo-1 और count-demo-2 यानि increase count में add hoga )
  tags = {
    "Name" = "Count-Demo-${count.index}"
  }
```

## Step-06: c6-outputs.tf
- for loop with List
- for loop with Map
- for loop with Map Advanced
```t

# Output - For Loop with List  (for loop के through aws_instance.myec2vm resources से created every resource का public_dns output ने print होगा ) 
output "for_output_list" {
  description = "For Loop with List"
  value = [for instance in aws_instance.myec2vm: instance.public_dns ]
}

# Output - For Loop with Map (map में हमने aws_instance.myec2vm के through created every resource (instance) की instance.id और then उनका instance.public_dns output में print hoga ) 
output "for_output_map1" {
  description = "For Loop with Map"
  value = {for instance in aws_instance.myec2vm: instance.id => instance.public_dns}
}

# Output - For Loop with Map Advanced 
output "for_output_map2" {
  description = "For Loop with Map - Advanced"
  value = {for c, instance in aws_instance.myec2vm: c => instance.public_dns}
}

# Output Legacy Splat Operator (latest) - Returns the List
# (यहाँ हम aws_instance.myec2vm resources के through created every resource (instance) का public_dns output में print hoga ) 
output "legacy_splat_instance_publicdns" {
  description = "Legacy Splat Expression"
  value = aws_instance.myec2vm.*.public_dns
}  

# Output Latest Generalized Splat Operator - Returns the List
# (यहाँ हम aws_instance.myec2vm resources के through created every resource (instance) का public_dns output में print hoga ) 
output "latest_splat_instance_publicdns" {
  description = "Generalized Splat Expression"
  value = aws_instance.myec2vm[*].public_dns
}
```

## Step-07: Execute Terraform Commands
```t
# Terraform Initialize
terraform init

# Terraform Validate
terraform validate

# Terraform Plan
terraform plan
Observations: 
1) play with Lists and Maps for instance_type

# Terraform Apply
terraform apply -auto-approve
Observations: 
1) Two EC2 Instances (Count = 2) of a Resource myec2vm will be created
2) Count.index will start from 0 and end with 1 for VM Names
3) Review outputs in detail (for loop with list, maps, maps advanced, splat legacy and splat latest)


# Output after Terraform Apply
Outputs:

for_output_list = [
  "ec2-3-110-158-4.ap-south-1.compute.amazonaws.com",
  "ec2-15-206-165-152.ap-south-1.compute.amazonaws.com",
]
for_output_map1 = {
  "i-0f26c19373935ee95" = "ec2-15-206-165-152.ap-south-1.compute.amazonaws.com"
  "i-0f911a2e158bffcc1" = "ec2-3-110-158-4.ap-south-1.compute.amazonaws.com"
}
for_output_map2 = {
  "0" = "ec2-3-110-158-4.ap-south-1.compute.amazonaws.com"
  "1" = "ec2-15-206-165-152.ap-south-1.compute.amazonaws.com"
}
latest_splat_instance_publicdns = [
  "ec2-3-110-158-4.ap-south-1.compute.amazonaws.com",
  "ec2-15-206-165-152.ap-south-1.compute.amazonaws.com",
]
legacy_splat_instance_publicdns = [
  "ec2-3-110-158-4.ap-south-1.compute.amazonaws.com",
  "ec2-15-206-165-152.ap-south-1.compute.amazonaws.com",
```

## Step-08: Terraform Comments
- Single Line Comments - `#` and `//`
- Multi-line Commnets - `Start with /*` and `end with */`
- We are going to comment the legacy splat operator, which might be decommissioned in future versions
```t
# Output Legacy Splat Operator (latest) - Returns the List
/* output "legacy_splat_instance_publicdns" {
  description = "Legacy Splat Expression"
  value = aws_instance.myec2vm.*.public_dns
}  */
```

## Step-09: Clean-Up
```t
# Terraform Destroy
terraform destroy -auto-approve

# Files
rm -rf .terraform*
rm -rf terraform.tfstate*
```

