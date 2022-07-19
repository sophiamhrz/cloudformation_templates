# cloudformation_templates
AWS CloudFormation: Create a VPC with EC2 Instance

AWS CloudFormation, an Infrastructure as Code service, includes a template made up of nine sections. Although made up of nine sections, the Resources section is the only one required. For this project we will be using Mappings, Resources, and Outputs. I’ve broken my template down to explain what is going on in each section, but if you want to jump straight to the full CloudFormation Template feel free to skip to the end.

Mappings
The Mappings section is basically a lookup table using key: value relationships. In this template the Amazon Machine Image is being mapped to its respective Region. Because I only have two Regions mapped, the Template is restricted to these two regions and would fail if launched outside of these Regions.


AWS CloudFormation User Guide: Mappings

Mappings:
  RegionMap:
    us-east-1:
     "AMI": "ami-0742b4e673072066f"
    us-west-2:
     "AMI": "ami-0518bb0e75d3619ca"

Resources
The Resources section includes all the AWS resources that you want to create in the stack.


AWS CloudFormation User Guide: Resources
For VPC, the template is assigning a CIDR of 10.0.0.0/16, Enabling DNS, and giving the VPC a tag of Name: LUIT Project.
For InternetGateway, the template is creating the IGW and assigning a tag of Name: LUIT Project.
For InternetGatewayAttachment, the template is attaching the IGW to the VPC. This is the first time in the template that we are using the intrinsic function Ref, which returns the value of the specified parameter or resource. As an example for this template for InternetGatewayId, we are referencing the InternetGateway Logical Id and for the VpcId, we are referencing the VPC Logical Id.

4. For PublicSubnet1, the template is creating a public subnet.

For VpdId it’s using the Ref function to reference the VPC.
For AvailabilityZone it’s using the Select function to select a single object from a list. In this case it is returning the first Availability Zone using the GetAZ function.
CidrBlock assigns this subnet a CIDR of 10.0.1.0/24
MapPublicIpOnLaunch assigns a public IP on launch when set to true
Tags assigns a tag to this subnet.
5. For PublicSubnet2, the template everything is similar to PublicSubnet1 except for:

AvailabilityZone the template is now selecting the second AZ using the GetAZ function.
CidrBlock is set to 10.0.2.0/24
Tags
6. PrivateSubnet1 & PrivateSubnet2 follow the same structure as the public subnets with some differences.

AvailabilityZones will match respectively with their public counterparts. This ensures we have a public and private subnet in the same AZ.
CidrBlock will be different to ensure the subnets don’t overlap. I’ve chosen to use 10.0.11.0/24 and 10.0.12.0/24 to clearly differentiate the public subnets CIDR from the Private CIDR.
MapPublicIpOnLaunch is set to false. Since these are private subnets we don’t want to assign a public IP.

7. For PublicRouteTable, the template is creating a public route table named LUIT Project Public Routes referencing VPC for the VpcId.

8. For DefaultPublicRoute, the template is creating a route to the Internet Gateway for 0.0.0.0/0.

This is the first time in the template we use the DependsOn attribute. This specifies that the resource should follow the creation of another. In this case, the DefaultPublicRoute should follow the creation of the PublicRouteTable.
RouteTableId references the PublicRouteTable.
The GatewayId references the InternetGateway
9. For PublicSubnet1RouteTableAssociation, the template is associating PublicSubnet1 to the PublicRouteTable.

10. For PublicSubnet2RouteTableAssociation, the template is associating PublicSubnet2 to the PublicRouteTable.

You’ll notice that the template has not created a default route table or associated the two private subnets. This is because a default route table is created with the VPC and all subnets are automatically associated unless otherwise specified. For this reason we don’t need to add them to the template since its the default.


11. For WebserverSecurityGroup, the template is creating a security group with an inbound rule allowing all traffic on HTTP.


12. For EC2Instance, the template is creating a single EC2 Instance with the following properties:

ImageID: You could directly put in the AMI-Id for this property, but since we want the ability to use the template in two regions we use the FindInMap function to reference our Mappings.
InstanceType: set to t2.micro.
SubnetId: references PublicSubnet1 to launch the instance.
SecurityGroupIds: Uses the references the WebServerSecurityGroup to associate it with the instance.
Tags
UserData: This section is used to bootstrap an instance. This particular template is running an update, installing an Apache web server and creating a simple web page. It uses the Sub function to update the AWS::Region pseudo parameter for the actual region the stack is created in. It uses the Base64 function to pass encoded data to EC2 instances.

Outputs
The Outputs section prints information to the Outputs tab in Cloudformation after the stack is created and can also be used to import information into other stacks. A basic example is the one used in this template where the Output simple displays the Public IP address of the created instance using the GetAtt function.


AWS CloudFormation User Guide: Outputs

For more detailed explanations of CloudFormation I recommend you review the AWS CloudFormation User Guide. This resource has crucial for me while diving into CloudFormation. I hope you found this valuable. Feel free to use the provided template to save yourself some time. For more Cloudformation Templates, check out my GitHub.

Full CloudFormation Template:

