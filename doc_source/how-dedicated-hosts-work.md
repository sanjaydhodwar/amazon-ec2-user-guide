# Working with Dedicated Hosts<a name="how-dedicated-hosts-work"></a>

To use a Dedicated Host, you first allocate hosts for use in your account\. You then launch instances onto the hosts by specifying *host* tenancy for the instance\. You must select a specific host for the instance to launch on to, or you can allow it to launch on to any host that has auto\-placement enabled and matches its instance type\. When an instance is stopped and restarted, the *Host affinity* setting determines whether it's restarted on the same, or a different, host\.

If you no longer need an On\-Demand host, you can stop the instances running on the host, direct them to launch on a different host, and then *release* the host\.

**Topics**
+ [Understanding Auto\-Placement and Affinity](#dedicated-hosts-understanding)
+ [Allocating Dedicated Hosts](#dedicated-hosts-allocating)
+ [Launching Instances onto Dedicated Hosts](#launching-dedicated-hosts-instances)
+ [Modifying Dedicated Host Auto\-Placement](#modify-host-auto-placement)
+ [Modifying Instance Tenancy and Affinity](#moving-instances-dedicated-hosts)
+ [Viewing Dedicated Hosts](#dedicated-hosts-managing)
+ [Tagging Dedicated Hosts](#dedicated-hosts-tagging)
+ [Monitoring Dedicated Hosts](#dedicated-hosts-monitoring)
+ [Releasing Dedicated Hosts](#dedicated-hosts-releasing)
+ [Purchasing Dedicated Host Reservations](#purchasing-dedicated-host-reservations)
+ [Viewing Dedicated Host Reservations](#viewing-host-reservations)
+ [Tagging Dedicated Host Reservations](#tagging-host-reservations)

## Understanding Auto\-Placement and Affinity<a name="dedicated-hosts-understanding"></a>

Placement control happens on both the instance level and host level\.

### Auto\-Placement<a name="dedicated-hosts-auto-placement"></a>

Auto\-placement allows you to manage whether instances that you launch are launched onto a specific host, or onto any available host that has matching configurations\. Auto\-placement must be configured at the host level\.

When a Dedicated Host's auto\-placement is *disabled*, it only accepts *Host* tenancy instance launches that specify its unique host ID\. This is the default setting for new Dedicated Hosts\.

When a Dedicated Host's auto\-placement is *enabled*, it accepts any untargeted instance launches that match its instance type configuration\.

When launching an instance, you need to configure its tenancy\. Launching an instance onto a Dedicated Host without providing a specific `HostId`, enables it to launch on any Dedicated Host that has auto\-placement *enabled* and matches its instance type\.

### Host Affinity<a name="dedicated-hosts-affinity"></a>

Host Affinity is configured at the instance level\. It establishes a launch relationship between an instance and a Dedicated Host\.

When affinity is set to `Host`, an instance launched onto a specific host always restarts on the same host if stopped\. This applies to both targeted and untargeted launches\.

When affinity is set to `Off`, and you stop and restart the instance, it can be restarted on any available host\. However, it tries to launch back onto the last Dedicated Host on which it ran \(on a best\-effort basis\)\.

## Allocating Dedicated Hosts<a name="dedicated-hosts-allocating"></a>

To begin using Dedicated Hosts, they must be allocated to your account\. You can allocate Dedicated Hosts to your account using the Amazon EC2 console or the command line tools\.

**To allocate Dedicated Hosts using the Amazon EC2 console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Dedicated Hosts**, **Allocate Dedicated Host**\.

1. Configure the following Dedicated Host options:

   1. **Instance type**—The type of instance that you want to launch on the Dedicated Host\.

   1. **Availability Zone**—The Availability Zone in which the Dedicated Host is located\.

   1. To allow the Dedicated Host to accept untargeted instance launches that match its instance type, for **Instance auto\-placement**, choose **Enable**\.

      For more information about auto\-placement, see [Understanding Auto\-Placement and Affinity](#dedicated-hosts-understanding)\.

   1. To enable host recovery for the Dedicated Host, for **Host recovery** choose **Enable**\. For more information, see [Host Recovery](dedicated-hosts-recovery.md)\.

   1. **Quantity**—The number of Dedicated Hosts to allocate with these options\.

1. \(Optional\) Choose **Add Tag** and enter a tag key and a tag value\.

1. Choose **Allocate host**\.

**To allocate Dedicated Hosts using the command line tools**

Use one of the following commands\. The following commands allocate a Dedicated Host that supports *untargeted* `m4.large` instance launches in the `eu-west-1a` Availability Zone, enable host recovery, and apply a tag with a key of `purpose` and a value of `production`\.
+ [allocate\-hosts](https://docs.aws.amazon.com/cli/latest/reference/ec2/allocate-hosts.html) \(AWS CLI\)

  ```
  aws ec2 allocate-hosts --instance-type "m4.large" --availability-zone "eu-west-1a" --auto-placement "off" --host-recovery "on" --quantity 1 --tag-specifications 'ResourceType=dedicated-host,Tags=[{Key=purpose,Value=production}]'
  ```
+ [New\-EC2Host](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2Host.html) \(AWS Tools for Windows PowerShell\)

  The `TagSpecification` parameter used to tag a Dedicated Host on creation requires an object that specifies the type of resource to be tagged, the tag key, and the tag value\. The following commands create the required object\.

  ```
  PS C:\> $tag = @{ Key="purpose"; Value="production" }
  PS C:\> $tagspec = new-object Amazon.EC2.Model.TagSpecification
  PS C:\> $tagspec.ResourceType = "dedicated-host"
  PS C:\> $tagspec.Tags.Add($tag)
  ```

  The following command allocates the Dedicated Host and applies the tag specified in the `$tagspec` object\.

  ```
  PS C:\> New-EC2Host -InstanceType m4.large -AvailabilityZone eu-west-1a -AutoPlacement Off -HostRecovery On -Quantity 1 -TagSpecification $tagspec
  ```

The Dedicated Host capacity is made available in your account immediately\.

If you launch instances with `host` tenancy but do not have any active Dedicated Host in your account, you receive an error and the instance launch fails\.

## Launching Instances onto Dedicated Hosts<a name="launching-dedicated-hosts-instances"></a>

After you have allocated a Dedicated Host, you can launch instances onto it\. You cannot launch instances with `host` tenancy if you do not have active Dedicated Hosts with enough available capacity for the instance type that you are launching\.

**Note**  
The instances launched onto Dedicated Hosts can only be launched in a VPC\. For more information, see [Introduction to VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Introduction.html)\.

Before you launch your instances, take note of the limitations\. For more information, see [Dedicated Hosts Limitations and Restrictions](dedicated-hosts-overview.md#dedicated-hosts-limitations)\.

**To launch an instance onto a specific Dedicated Host from the Dedicated Hosts page**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Choose **Dedicated Hosts** in the navigation pane\.

1. On the **Dedicated Hosts** page, select a host choose **Actions**, **Launch Instance\(s\) onto Host**\.

1. Select an AMI from the list\. Windows, SUSE, and RHEL AMIs provided by Amazon EC2 can't be used with Dedicated Hosts\.

1. On the **Choose an Instance Type** page, keep the instance type that is selected by default, and then choose **Next: Configure Instance Details**\. 

   The instance type is determined by the host that you selected\.

1. On the **Configure Instance Details** page, configure the instance settings to suit your needs, and then for **Affinity**, choose one of the following options:
   + **Off**—The instance launches onto the specified host, but it is not guaranteed to restart on the same Dedicated Host if stopped\.
   + **Host**—If stopped, the instance always restarts on this specific host\.

   For more information about Affinity, see [Understanding Auto\-Placement and Affinity](#dedicated-hosts-understanding)\.
**Note**  
The **Tenancy** and **Host** options are pre\-configured based on the host that you selected\.

1. Choose **Review and Launch**\.

1. On the **Review Instance Launch** page, choose **Launch**\.

1. When prompted, select an existing key pair or create a new one, and then choose **Launch Instances**\.

**To launch an instance onto a Dedicated Host using the Launch Instance wizard**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Instances**, **Launch Instance**\.

1. Select an AMI from the list\. Windows, SUSE, and RHEL AMIs provided by Amazon EC2 can't be used with Dedicated Hosts\.

1. Select the type of instance to launch and choose **Next: Configure Instance Details**\.

1. On the **Configure Instance Details** page, configure the instance settings to suit your needs, and then configure the following Dedicated Host\-specific settings:
   + Tenancy—Choose **Dedicated Host \- Launch this instance on a Dedicated Host**\.
   + Host—Choose either **Use auto\-placement** to launch the instance on any Dedicated Host that has auto\-placement enabled, or select a specific Dedicated Host in the list\. If Dedicated Hosts does not support the selected instance type, it is disabled in the list\.
   + Affinity—Choose one of the following options:
     + **Off**—The instance launches onto the specified host, but it is not guaranteed to restart on it if stopped\.
     + **Host**—If stopped, the instance always restarts on the specified host\.

   For more information, see [Understanding Auto\-Placement and Affinity](#dedicated-hosts-understanding)\.
**Note**  
If you are unable to see these settings, check that you have selected a VPC in the **Network** menu\.

1. Choose **Review and Launch**\.

1. On the **Review Instance Launch** page, choose **Launch**\.

1. When prompted, select an existing key pair or create a new one, and then choose **Launch Instances**\.

**To launch an instance onto a Dedicated Host using the command line tools**

Use one of the following commands and specify the instance affinity, tenancy, and host in the `Placement` request parameter:
+ [run\-instances](https://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html) \(AWS CLI\)
+ [New\-EC2Instance](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2Instance.html) \(AWS Tools for Windows PowerShell\)

## Modifying Dedicated Host Auto\-Placement<a name="modify-host-auto-placement"></a>

You can modify a Dedicated Host's auto\-placement settings after you have allocated it to your AWS account\.

**To modify a Dedicated Host's auto\-placement using the Amazon EC2 console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Choose **Dedicated Hosts** in the navigation pane\.

1. On the **Dedicated Hosts** page, select a host and choose **Actions**, **Modify Auto\-Placement**\.

1. On the Modify Auto\-placement window, for **Allow instance auto\-placement**, choose **Yes** to enable auto\-placement, or choose **No** to disable auto\-placement\. For more information, see [Understanding Auto\-Placement and Affinity](#dedicated-hosts-understanding)\.

1. Choose **Save**\.

**To modify a Dedicated Host's auto\-placement using the command line tools**

Use one of the following commands\. The following examples enable auto\-placement for the specified Dedicated Host\.
+ [modify\-hosts](https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-hosts.html) \(AWS CLI\)

  ```
  aws ec2 modify-hosts --auto-placement on --host-ids h-012a3456b7890cdef
  ```
+ [Edit\-EC2Host](https://docs.aws.amazon.com/powershell/latest/reference/items/Edit-EC2Host.html) \(AWS Tools for Windows PowerShell\)

  ```
  PS C:\> Edit-EC2Host --AutoPlacement 1 --HostId h-012a3456b7890cdef
  ```

## Modifying Instance Tenancy and Affinity<a name="moving-instances-dedicated-hosts"></a>

You can change the tenancy of an instance from `dedicated` to `host`, or from `host` to `dedicated`, after you have launched it\.

**To modify instance tenancy and affinity using the Amazon EC2 console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Choose **Instances**, and select the instance to modify\.

1. Choose **Actions**, **Instance State**, and **Stop**\.

1. Open the context \(right\-click\) menu on the instance and choose **Instance Settings**, **Modify Instance Placement**\.

1. On the **Modify Instance Placement** page, configure the following:
   + **Tenancy**—Choose one of the following:
     + Run a dedicated hardware instance—Launches the instance as a Dedicated Instance\. For more information, see [Dedicated Instances](dedicated-instance.md)\.
     + Launch the instance on a Dedicated Host—Launches the instance onto a Dedicated Host with configurable affinity\.
   + **Affinity**—Choose one of the following:
     + This instance can run on any one of my hosts—The instance launches onto any available Dedicated Host in your account that supports its instance type\.
     + This instance can only run on the selected host—The instance is only able to run on the Dedicated Host selected for **Target Host**\.
   + **Target Host**—Select the Dedicated Host that the instance must run on\. If no target host is listed, you may not have available, compatible Dedicated Hosts in your account\.

   For more information, see [Understanding Auto\-Placement and Affinity](#dedicated-hosts-understanding)\.

1. Choose **Save**\.

**To modify instance tenancy and affinity using the command line tools**

Use one of the following commands\. The following examples change the specified instance's affinity from `default` to `host`, and specify the Dedicated Host that the instance has affinity with\.
+ [modify\-instance\-placement](https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-instance-placement.html) \(AWS CLI\)

  ```
  aws ec2 modify-instance-placement --instance-id i-1234567890abcdef0 --affinity host --host-id h-012a3456b7890cdef
  ```
+ [Edit\-EC2InstancePlacement](https://docs.aws.amazon.com/powershell/latest/reference/items/Edit-EC2InstancePlacement.html) \(AWS Tools for Windows PowerShell\)

  ```
  PS C:\> Edit-EC2InstancePlacement -InstanceId i-1234567890abcdef0 -Affinity host -HostId h-012a3456b7890cdef
  ```

## Viewing Dedicated Hosts<a name="dedicated-hosts-managing"></a>

You can view details about a Dedicated Host and the individual instances on it\.

**To view details of instances on a Dedicated Host using the Amazon EC2 console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Dedicated Hosts**\.

1. On the **Dedicated Hosts** page, select the host to view more information about\.

1. For information about the host, choose **Description**\. For information about instances running on the host, choose **Instances**\.

**To view details of instances on a Dedicated Host using the command line tools**

Use one of the following commands:
+ [describe\-hosts](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-hosts.html) \(AWS CLI\)

  ```
  aws ec2 describe-hosts --host-id host_id
  ```
+ [Get\-EC2Host](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-EC2Host.html) \(AWS Tools for Windows PowerShell\)

  ```
  PS C:\> Get-EC2Host -HostId host_id
  ```

## Tagging Dedicated Hosts<a name="dedicated-hosts-tagging"></a>

You can assign custom tags to your existing Dedicated Hosts to categorize them in different ways, for example, by purpose, owner, or environment\. This helps you to quickly find a specific Dedicated Host based on the custom tags that you assigned\. Dedicated Host tags can also be used for cost allocation tracking\.

You can also apply tags to Dedicated Hosts at the time of creation\. For more information, see [Allocating Dedicated Hosts](#dedicated-hosts-allocating)\.

You can tag a Dedicated Host using the Amazon EC2 console and command line tools\.

**To tag a Dedicated Host using the console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Dedicated Hosts**\. 

1. Select the Dedicated Host to tag, and then choose **Tags**\. 

1. Choose **Add/Edit Tags**\.

1. In the **Add/Edit Tags** dialog box, choose **Create Tag**, and then specify the key and value for the tag\.

1. \(Optional\) Choose **Create Tag** to add additional tags to the Dedicated Host\.

1. Choose **Save**\.

**To tag a Dedicated Host using the command line**

Use one of the following commands:
+ [create\-tags](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-tags.html) \(AWS CLI\)

  The following command tags the specified Dedicated Host with `Owner=TeamA`\.

  ```
  aws ec2 create-tags --resources h-abc12345678909876 --tags Key=Owner,Value=TeamA
  ```
+ [New\-EC2Tag](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2Tag.html) \(AWS Tools for Windows PowerShell\)

  The `New-EC2Tag` command needs a `Tag` object, which specifies the key and value pair to be used for the Dedicated Host tag\. The following commands create a `Tag` object named `$tag`, with a key and value pair of `Owner` and `TeamA` respectively\.

  ```
  PS C:\> $tag = New-Object Amazon.EC2.Model.Tag
  PS C:\> $tag.Key = "Owner"
  PS C:\> $tag.Value = "TeamA"
  ```

  The following command tags the specified Dedicated Host with the `$tag` object\.

  ```
  PS C:\> New-EC2Tag -Resource h-abc12345678909876 -Tag $tag
  ```

## Monitoring Dedicated Hosts<a name="dedicated-hosts-monitoring"></a>

Amazon EC2 constantly monitors the state of your Dedicated Hosts\. Updates are communicated on the Amazon EC2 console\. You can also obtain information about your Dedicated Hosts by using the command line tools\.

**To view the state of a Dedicated Host using the Amazon EC2 console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. In the navigation pane, choose **Dedicated Hosts**\.

1. Locate the Dedicated Host in the list and review the value in the **State** column\.

**To view the state of a Dedicated Host using the command line tools**

Use one of the following commands and then review the `state` property in the `hostSet` response element:
+ [describe\-hosts](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-hosts.html) \(AWS CLI\)

  ```
  aws ec2 describe-hosts --host-id host_id
  ```
+ [Get\-EC2Host](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-EC2Host.html) \(AWS Tools for Windows PowerShell\)

  ```
  PS C:\> Get-EC2Host -HostId host_id
  ```

The following table explains the possible Dedicated Host states\.


| **State** | **Description** | 
| --- | --- | 
| available | AWS hasn't detected an issue with the Dedicated Host\. No maintenance or repairs are scheduled\. Instances can be launched onto this Dedicated Host\. | 
| released | The Dedicated Host has been released\. The host ID is no longer in use\. Released hosts cannot be reused\. | 
| under\-assessment | AWS is exploring a possible issue with the Dedicated Host\. If action must be taken, you are notified via the AWS Management Console or email\. Instances cannot be launched onto a Dedicated Host in this state\. | 
| pending | A host recovery is in progress\. For more information, see [Host Recovery](dedicated-hosts-recovery.md)\. | 
| permanent\-failure | An unrecoverable failure has been detected\. You receive an eviction notice through your instances and by email\. Your instances might continue to run\. If you stop or terminate all instances on a Dedicated Host with this state, AWS retires the host\. AWS does not restart instances in this state\. Instances cannot be launched onto Dedicated Hosts in this state\. | 
| released\-permanent\-failure | AWS permanently releases Dedicated Hosts that have failed and no longer have running instances on them\. The Dedicated Host ID is no longer available for use\. | 

## Releasing Dedicated Hosts<a name="dedicated-hosts-releasing"></a>

Any running instances on the Dedicated Host must be stopped before you can release the host\. These instances can be migrated to other Dedicated Hosts in your account so that you can continue to use them\. These steps apply only to On\-Demand Dedicated Hosts\.

**To release a Dedicated Host using the Amazon EC2 console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Choose **Dedicated Hosts** in the navigation pane\.

1. On the **Dedicated Hosts** page, select the Dedicated Host to release\.

1. Choose **Actions**, **Release Hosts**\.

1. Choose **Release** to confirm\.

**To release a Dedicated Host using the command line tools**

Use one of the following commands:
+ [release\-hosts](https://docs.aws.amazon.com/cli/latest/reference/ec2/release-hosts.html) \(AWS CLI\)

  ```
  aws ec2 release-hosts --host-ids host_id
  ```
+ [Remove\-EC2Hosts](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-EC2Host.html) \(AWS Tools for Windows PowerShell\)

  ```
  PS C:\> Remove-EC2Hosts -HostId host_id
  ```

After you release a Dedicated Host, you cannot reuse the same host or host ID again, and you are no longer charged On\-Demand billing rates for it\. The Dedicated Host's state is changed to `released`, and you are not able to launch any instances onto that host\.

**Note**  
If you have recently released Dedicated Hosts, it may take some time for them to stop counting towards your limit\. During this time, you may experience `LimitExceeded` errors when trying to allocate new Dedicated Hosts\. If this is the case, try allocating new hosts again after a few minutes\.

The instances that were stopped are still available for use and are listed on the **Instances** page\. They retain their `host` tenancy setting\.

## Purchasing Dedicated Host Reservations<a name="purchasing-dedicated-host-reservations"></a>

You can purchase reservations using the Amazon EC2 console or command line tools\.

**To purchase reservations using the Amazon EC2 console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Choose **Dedicated Hosts**, **Dedicated Host Reservations**, **Purchase Dedicated Host Reservation**\.

1. On the **Purchase Dedicated Host Reservation** screen, you can search for available offerings using the default settings, or you can specify custom values for the following:
   + **Host instance family**—The options listed correspond with the Dedicated Hosts in your account that are not already assigned to a reservation\.
   + **Availability Zone**—The Availability Zone of the Dedicated Hosts in your account that aren't already assigned to a reservation\.
   + **Payment option**—The payment option for the offering\.
   + **Term**—The term of the reservation, which can be one or three years\.

1. Choose **Find offering** and select an offering that matches your requirements\.

1. Choose the Dedicated Hosts to associate with the reservation, and then choose **Review**\.

1. Review your order and choose **Purchase**\.

**To purchase reservations using the command line tools**

1. Use one of the following commands to list the available offerings that match your needs\. The following examples list the offerings that support instances in the `m4` instance family and have a one\-year term\.
**Note**  
The term is specified in seconds\. A one\-year term includes 31536000 seconds, and a three\-year term includes 94608000 seconds\.
   + [describe\-host\-reservation\-offerings](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-host-reservation-offerings.html) \(AWS CLI\)

     ```
     aws ec2 describe-host-reservation-offerings --filter Name=instance-family,Values=m4 --max-duration 31536000
     ```
   + [Get\-EC2HostReservationOffering](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-EC2HostReservationOffering.html) \(AWS Tools for Windows PowerShell\)

     ```
     PS C:\> $filter = @{Name="instance-family"; Value="m4"}
     ```

     ```
     PS C:\> Get-EC2HostReservationOffering -filter $filter -MaxDuration 31536000
     ```

   Both commands return a list of offerings that match your criteria\. Note the `offeringId` of the offering to purchase\. 

1. Use one of the following commands to purchase the offering and provide the `offeringId` noted in the previous step\. The following examples purchase the specified reservation and associate it with a specific Dedicated Host that is already allocated in the AWS account\.
   + [purchase\-host\-reservation](https://docs.aws.amazon.com/cli/latest/reference/ec2/purchase-host-reservation.html) \(AWS CLI\)

     ```
     aws ec2 purchase-host-reservation --offering-id hro-03f707bf363b6b324 --host-id-set h-013abcd2a00cbd123
     ```
   + [New\-EC2HostReservation](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2HostReservation.html) \(AWS Tools for Windows PowerShell\)

     ```
     PS C:\> New-EC2HostReservation -OfferingId hro-03f707bf363b6b324 -HostIdSet h-013abcd2a00cbd123
     ```

## Viewing Dedicated Host Reservations<a name="viewing-host-reservations"></a>

You can view information about the Dedicated Hosts associated with your reservation, including:
+ The term of the reservation
+ The payment option
+ The start and end dates

**To view details of reservations using the Amazon EC2 console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. Choose **Dedicated Hosts** in the navigation pane\.

1. On the **Dedicated Hosts** page, choose **Dedicated Host Reservations**, and then select the reservation from the list provided\.

1. Choose **Details** for information about the reservation\.

1. Choose **Hosts** for information about the Dedicated Hosts with which the reservation is associated\.

**To view details of reservations using the command line tools**

Use one of the following commands:
+ [describe\-host\-reservations](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-host-reservations.html) \(AWS CLI\)

  ```
  aws ec2 describe-host-reservations
  ```
+ [Get\-EC2HostReservation](https://docs.aws.amazon.com/powershell/latest/reference/items/Get-EC2HostReservation.html) \(AWS Tools for Windows PowerShell\)

  ```
  PS C:\> Get-EC2HostReservation
  ```

## Tagging Dedicated Host Reservations<a name="tagging-host-reservations"></a>

You can assign custom tags to your Dedicated Host Reservations to categorize them in different ways, for example, by purpose, owner, or environment\. This helps you to quickly find a specific Dedicated Host Reservation based on the custom tags you assigned\.

You can tag a Dedicated Host Reservation using the AWS CLI only\.

**To tag a Dedicated Host Reservation using the command line**

Use one of the following commands:
+ [create\-tags](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-tags.html) \(AWS CLI\)

  ```
  aws ec2 create-tags --resources hr-1234563a4ffc669ae --tags Key=Owner,Value=TeamA
  ```
+ [New\-EC2Tag](https://docs.aws.amazon.com/powershell/latest/reference/items/New-EC2Tag.html) \(AWS Tools for Windows PowerShell\)

  The `New-EC2Tag` command needs a `Tag` parameter, which specifies the key and value pair to be used for the Dedicated Host Reservation tag\. The following commands create the `Tag` parameter\.

  ```
  PS C:\> $tag = New-Object Amazon.EC2.Model.Tag
  PS C:\> $tag.Key = "Owner"
  PS C:\> $tag.Value = "TeamA"
  ```

  ```
  PS C:\> New-EC2Tag -Resource hr-1234563a4ffc669ae -Tag $tag
  ```