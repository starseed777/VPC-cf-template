This is a cloud formation template that will automate a VPC and various other resources in AWS. Refer to the JSON code in this repository for the in depth explanations of what each block of code will deploy. Since we are doing this in an automated fashion we will have to recreate the manual processes we would normally do in the AWS console into our code (this will help for visualization purposes incase you get confused!)


- Line 5-15: We will be using parameters for our cloudformation template to be able to specify a keypair and an AMI for when the stack gets created. Notice how when creating a stack in the cloudformation console we have to specify a keypair and an AMI before everything else? Same process but we are incorporating that into our code for automation.


- Line 16: This is where we begin to specify the individual resources that will be deployed.


- Line 17-33: This will deploy the first VPC with its configurations such as cidr block, DNS hostname , DNS support, instance tenancy and tags. As you can see DNS hostname and DNS support will be enabled in these configurations which will allow AWS to provide  instances in the VPC with public DNS hostnames. Instances with a public IP address receive corresponding public DNS hostnames. As for instance tenancy we will just be using the default tenancy (shared tenancy) where instances from different AWS customers may reside on the same piece of physical hardware. As for the tags this is just for naming the VPC, in this case it will be named "VPC1" (key value).

- Line 34-52: This block of code will deploy a public subnet in the first VPC. On "VpcId" we are making a reference to the logical ID of our first VPC that way this subnet will be created in the first VPC. For our tags we will just be naming this "vpc1subnet_public" (key value). Now for "MapPublicIpOnLaunch" this will be enabled which will allow a public IP address to be automatically assigned to an instance launched with this subnet. The availability zone option will have a loop that will iterate through all AZ options and select the first index - hence the 0. This loop will allow for us to automate the process and not have to hard code the AZ's into our template. The function "Fn::Select" returns a single object from a list of objects by index. "Fn::GetAZs" function returns all Availability Zones for a region. Lastly the cidr block this will be our range of ipv4 addresses in the form of classless inter-domain routing.

- Line 53-71: This block will deploy a private subnet with the same reference to the logical ID of our first VPC. However since it is going to be private we will not be enabling the "MapPublicIpOnLaunch" therefore no public IP address is assigned. As mentioned earlier on how we are selecting our AZ and lastly our cidr block. Keep in mind we cannot have our cidr blocks overlapping. 

- Line 72-78: This block will deploy an internet gateway, it will be for our first VPC therefore in the code we have "DependsOn": "Vpc1". As for the properties we have the tag name which will be "Vpc1igw".

- Line 79-86: We previously made our internet gateway itself but now we have to attach it to our first VPC. As you can see the logical ID of our gateway attachment is on line 79 "igwattachmentvpc1". In this case as you can see the "Type" is now "AWS::EC2::VPCGatewayAttachment". The internet gateway that we previously created is being refered to as well as our first VPC in order for our gateway to attach to the first VPC.

- Line 87-101: This code block will deploy a second VPC. the logical ID of this one will be "Vpc2", We will be using a different cidr block for this VPC but we will have the same DNS hostname / DNS support / instance tenancy configurations as our first VPC. As for our tags the keyname value for this VPC will be "Vpc2".

- Line 102-121: This block of code will deploy a public subnet to our second VPC. The logical ID of this public subnet will be "Vpc2snA1". In order to have this subnet in our second VPC, we will have to refer to the logical ID of our second VPC. For the tags we will follow the same naming covention as seen previously but will instead name it "vpc2subnet_public". As for "MapPublicIpOnLaunch" that will be enabled + for our AZ selection same kind of iteration (explanation already mentioned previously for earlier public subnet in first VPC). Lastly our cidr block- remember we have to have our cidr within the same range as our VPC. 

- Line 132-139: This block of code will deploy a private subnet to our second VPC. As you can see we will have a different logical ID. We will be making the same reference to our second VPC's logical ID that way this subnet will be deployed to our second VPC. We will have different tags on this one as well - "vpc2subnet_private". This subnet is private therefore we will not enable "MapPublicIpOnLaunch".

- Line 140-146: This will deploy another internet gateway but will be for our second VPC hence the "DependsOn": "Vpc2". Tags will be the standard naming covention --> "Vpc2igw"

- Line 147-154: This will be the internet gateway attachment for the second internet gateway that will be attached to our second VPC.

- Line 155-161: This will be the public route table for the first VPC, as you can see the reference to the logcial ID of the first VPC. For the tags we will have the name key value as "RT-Public-VPC1".

- Line 162-170 Since we have the route table set up this is where we specify the path. This depends on our first internet gateway as you can see from "DependsOn": "igwattachmentvpc1". The destination this route will be going is 0.0.0.0/0 aka to all IPv4 addresses. The route table ID and gateway ID are referring to the logical ID of the public route table from earlier + the first internet gateway that was deployed. 