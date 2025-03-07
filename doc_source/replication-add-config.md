# Replication configuration<a name="replication-add-config"></a>

Amazon S3 stores a replication configuration as XML\. In the replication configuration XML file, you specify an AWS Identity and Access Management \(IAM\) role and one or more rules\. 

```
<ReplicationConfiguration>
    <Role>IAM-role-ARN</Role>
    <Rule>
        ...
    </Rule>
    <Rule>
         ... 
    </Rule>
     ...
</ReplicationConfiguration>
```

Amazon S3 can't replicate objects without your permission\. You grant permissions with the IAM role that you specify in the replication configuration\. Amazon S3 assumes the IAM role to replicate objects on your behalf\. You must grant the required permissions to the IAM role first\. For more information about managing permissions, see [Setting up permissions](setting-repl-config-perm-overview.md)\.

You add one rule in a replication configuration in the following scenarios:
+ You want to replicate all objects\.
+ You want to replicate one subset of objects\. You identify the object subset by adding a filter in the rule\. In the filter, you specify an object key prefix, tags, or a combination of both, to identify the subset of objects that the rule applies to\. 

You add multiple rules in a replication configuration if you want to replicate different subsets of objects\. In each rule, you specify a filter that selects a different subset of objects\. For example, you might choose to replicate objects that have either `tax/` or `document/` key prefixes\. To do this, you add two rules, one that specifies the `tax/` key prefix filter and another that specifies the `document/` key prefix\.

The following sections provide additional information\.

**Topics**
+ [Basic rule configuration](#replication-config-min-rule-config)
+ [Optional: Specifying a filter](#replication-config-optional-filter)
+ [Additional destination configurations](#replication-config-optional-dest-config)
+ [Example replication configurations](#replication-config-example-configs)
+ [Backward compatibility](#replication-backward-compat-considerations)

## Basic rule configuration<a name="replication-config-min-rule-config"></a>

### <a name="replication-config-min-rule-config"></a>

Each rule must include the rule's status and priority\. The rule must also indicate whether to replicate delete markers\. 
+ `Status` indicates whether the rule is enabled or disabled by using the values `Enabled` or `Disabled`\. If a rule is disabled, Amazon S3 doesn't perform the actions specified in the rule\. 
+ `Priority` indicates which rule has precedence whenever two or more replication rules conflict\. Amazon S3 attempts to replicate objects according to all replication rules\. However, if there are two or more rules with the same destination bucket, then objects are replicated according to the rule with the highest priority\. The higher the number, the higher the priority\.
+ `DeleteMarkerReplication` indicates whether to replicate delete markers by using the values `Enabled` or `Disabled`\.

In the destination configuration, you must provide the name of the bucket or buckets where you want Amazon S3 to replicate objects\. 

The following example shows the minimum requirements for a V2 rule\. For backward compatibility, Amazon S3 continues to support the XML V1 format\. For more information, see [Backward compatibility](#replication-backward-compat-considerations)\.

```
...
    <Rule>
        <ID>Rule-1</ID>
        <Status>Enabled-or-Disabled</Status>
        <Filter>
            <Prefix></Prefix>   
        </Filter>
        <Priority>integer</Priority>
        <DeleteMarkerReplication>
           <Status>Enabled-or-Disabled</Status>
        </DeleteMarkerReplication>
        <Destination>        
           <Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET</Bucket> 
        </Destination>    
    </Rule>
    <Rule>
         ...
    </Rule>
     ...
...
```

You can also specify other configuration options\. For example, you might choose to use a storage class for object replicas that differs from the class for the source object\. 

## Optional: Specifying a filter<a name="replication-config-optional-filter"></a>

To choose a subset of objects that the rule applies to, add an optional filter\. You can filter by object key prefix, object tags, or a combination of both\. If you filter on both a key prefix and object tags, Amazon S3 combines the filters by using a logical `AND` operator\. In other words, the rule applies to a subset of objects with a specific key prefix and specific tags\. 

**Filter based on object key prefix**  
To specify a rule with a filter based on an object key prefix, use the following code\. You can specify only one prefix\.

```
<Rule>
    ...
    <Filter>
        <Prefix>key-prefix</Prefix>   
    </Filter>
    ...
</Rule>
...
```

**Filter based on object tags**  
To specify a rule with a filter based on object tags, use the following code\. You can specify one or more object tags\.

```
<Rule>
    ...
    <Filter>
        <And>
            <Tag>
                <Key>key1</Key>
                <Value>value1</Value>
            </Tag>
            <Tag>
                <Key>key2</Key>
                <Value>value2</Value>
            </Tag>
             ...
        </And>
    </Filter>
    ...
</Rule>
...
```

**Filter with a key prefix and object tags**  
To specify a rule filter with a combination of a key prefix and object tags, use the following code\. You wrap these filters in an `And` parent element\. Amazon S3 performs a logical `AND` operation to combine these filters\. In other words, the rule applies to a subset of objects with both a specific key prefix and specific tags\. 

```
<Rule>
    ...
    <Filter>
        <And>
            <Prefix>key-prefix</Prefix>
            <Tag>
                <Key>key1</Key>
                <Value>value1</Value>
            </Tag>
            <Tag>
                <Key>key2</Key>
                <Value>value2</Value>
            </Tag>
             ...
    </Filter>
    ...
</Rule>
...
```

**Note**  
If you specify a rule with an empty filter tag, your rule applies to all objects in your bucket\.

## Additional destination configurations<a name="replication-config-optional-dest-config"></a>

In the destination configuration, you specify the bucket or buckets where you want Amazon S3 to replicate objects\. You can set configurations to replicate objects from one source bucket to one or more destination buckets\. 

```
...
<Destination>        
    <Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET</Bucket>
</Destination>
...
```

You can add the following options in the `<Destination>` element\.

**Topics**
+ [Specify storage class](#storage-class-configuration)
+ [Add multiple destination buckets](#multiple-destination-buckets-configuration)
+ [Specify different parameters for each replication rule with multiple destination buckets](#replication-rule-configuration)
+ [Change replica ownership](#replica-ownership-configuration)
+ [Enable S3 Replication Time Control](#rtc-configuration)
+ [Replicate objects created with server\-side encryption by using AWS KMS](#sse-kms-configuration)

### Specify storage class<a name="storage-class-configuration"></a>

You can specify the storage class for the object replicas\. By default, Amazon S3 uses the storage class of the source object to create object replicas, as in the following example\.

```
...
<Destination>
       <Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET</Bucket>
       <StorageClass>storage-class</StorageClass>
</Destination>
...
```

### Add multiple destination buckets<a name="multiple-destination-buckets-configuration"></a>

You can add multiple destination buckets in a single replication configuration, as follows\.

```
...
<Rule>
    <ID>Rule-1</ID>
    <Status>Enabled-or-Disabled</Status>
    <Priority>integer</Priority>
    <DeleteMarkerReplication>
       <Status>Enabled-or-Disabled</Status>
    </DeleteMarkerReplication>
    <Destination>        
       <Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET1</Bucket> 
    </Destination>    
</Rule>
<Rule>
    <ID>Rule-2</ID>
    <Status>Enabled-or-Disabled</Status>
    <Priority>integer</Priority>
    <DeleteMarkerReplication>
       <Status>Enabled-or-Disabled</Status>
    </DeleteMarkerReplication>
    <Destination>        
       <Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET2</Bucket> 
    </Destination>    
</Rule>
...
```

### Specify different parameters for each replication rule with multiple destination buckets<a name="replication-rule-configuration"></a>

When adding multiple destination buckets in a single replication configuration, you can specify different parameters for each replication rule, as follows\.

```
...
<Rule>
    <ID>Rule-1</ID>
    <Status>Enabled-or-Disabled</Status>
    <Priority>integer</Priority>
    <DeleteMarkerReplication>
       <Status>Disabled</Status>
    </DeleteMarkerReplication>
      <Metrics>
    <Status>Enabled</Status>
    <EventThreshold>
      <Minutes>15</Minutes> 
    </EventThreshold>
  </Metrics>
    <Destination>        
       <Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET1</Bucket> 
    </Destination>    
</Rule>
<Rule>
    <ID>Rule-2</ID>
    <Status>Enabled-or-Disabled</Status>
    <Priority>integer</Priority>
    <DeleteMarkerReplication>
       <Status>Enabled</Status>
    </DeleteMarkerReplication>
      <Metrics>
    <Status>Enabled</Status>
    <EventThreshold>
      <Minutes>15</Minutes> 
    </EventThreshold>
  </Metrics>
  <ReplicationTime>
    <Status>Enabled</Status>
    <Time>
      <Minutes>15</Minutes>
    </Time>
  </ReplicationTime>
    <Destination>        
       <Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET2</Bucket> 
    </Destination>    
</Rule>
...
```

### Change replica ownership<a name="replica-ownership-configuration"></a>

When the source and destination buckets aren't owned by the same accounts, you can change the ownership of the replica to the AWS account that owns the destination bucket\. To do so, add the `AccessControlTranslation` element\. This element takes the value `Destination`\.

```
...
<Destination>
   <Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET</Bucket>
   <Account>destination-bucket-owner-account-id</Account>
   <AccessControlTranslation>
       <Owner>Destination</Owner>
   </AccessControlTranslation>
</Destination>
...
```

If you don't add the `AccessControlTranslation` element to the replication configuration, the replicas are owned by the same AWS account that owns the source object\. For more information, see [Changing the replica owner](replication-change-owner.md)\.

### Enable S3 Replication Time Control<a name="rtc-configuration"></a>

You can enable S3 Replication Time Control \(S3 RTC\) in your replication configuration\. S3 RTC replicates most objects in seconds and 99\.99 percent of objects within 15 minutes \(backed by a service\-level agreement\)\. 

**Note**  
Only a value of `<Minutes>15</Minutes>` is accepted for `EventThreshold` and `Time`\.

```
...
<Destination>
  <Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET</Bucket>
  <Metrics>
    <Status>Enabled</Status>
    <EventThreshold>
      <Minutes>15</Minutes> 
    </EventThreshold>
  </Metrics>
  <ReplicationTime>
    <Status>Enabled</Status>
    <Time>
      <Minutes>15</Minutes>
    </Time>
  </ReplicationTime>
</Destination>
...
```

For more information, see [Meeting compliance requirements using S3 Replication Time Control \(S3 RTC\)](replication-time-control.md)\. For API examples, see [PutBucketReplication](https://docs.aws.amazon.com/AmazonS3/latest/API/API_PutBucketReplication.html) in the *Amazon Simple Storage Service API Reference*\.

### Replicate objects created with server\-side encryption by using AWS KMS<a name="sse-kms-configuration"></a>

Your source bucket might contain objects that were created with server\-side encryption by using AWS Key Management Service \(AWS KMS\) keys \(SSE\-KMS\)\. By default, Amazon S3 doesn't replicate these objects\. You can optionally direct Amazon S3 to replicate these objects\. To do so, first explicitly opt into this feature by adding the `SourceSelectionCriteria` element\. Then provide the AWS KMS key \(for the AWS Region of the destination bucket\) to use for encrypting object replicas\. The following example shows how to specify these elements\.

```
...
<SourceSelectionCriteria>
  <SseKmsEncryptedObjects>
    <Status>Enabled</Status>
  </SseKmsEncryptedObjects>
</SourceSelectionCriteria>
<Destination>
  <Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET</Bucket>
  <EncryptionConfiguration>
    <ReplicaKmsKeyID>AWS KMS key ID to use for encrypting object replicas</ReplicaKmsKeyID>
  </EncryptionConfiguration>
</Destination>
...
```

For more information, see [Replicating objects created with server\-side encryption \(SSE\) using KMS keys](replication-config-for-kms-objects.md)\.

## Example replication configurations<a name="replication-config-example-configs"></a>

To get started, you can add the following example replication configurations to your bucket, as appropriate\.

**Important**  
To add a replication configuration to a bucket, you must have the `iam:PassRole` permission\. This permission allows you to pass the IAM role that grants Amazon S3 replication permissions\. You specify the IAM role by providing the Amazon Resource Name \(ARN\) that is used in the `Role` element in the replication configuration XML\. For more information, see [Granting a User Permissions to Pass a Role to an AWS service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_passrole.html) in the *IAM User Guide*\.

**Example 1: Replication configuration with one rule**  
The following basic replication configuration specifies one rule\. The rule specifies an IAM role that Amazon S3 can assume and a single destination bucket for object replicas\. The `Status` value of `Enabled` indicates that the rule is in effect\.  

```
<?xml version="1.0" encoding="UTF-8"?>
<ReplicationConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <Role>arn:aws:iam::account-id:role/role-name</Role>
  <Rule>
    <Status>Enabled</Status>

    <Destination><Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET</Bucket></Destination>

  </Rule>
</ReplicationConfiguration>
```
To choose a subset of objects to replicate, you can add a filter\. In the following configuration, the filter specifies an object key prefix\. This rule applies to objects that have the prefix `Tax/` in their key names\.   

```
<?xml version="1.0" encoding="UTF-8"?>
<ReplicationConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <Role>arn:aws:iam::account-id:role/role-name</Role>
  <Rule>
    <Status>Enabled</Status>
    <Priority>1</Priority>
    <DeleteMarkerReplication>
       <Status>string</Status>
    </DeleteMarkerReplication>

    <Filter>
       <Prefix>Tax/</Prefix>
    </Filter>

    <Destination><Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET</Bucket></Destination>

  </Rule>
</ReplicationConfiguration>
```
If you specify the `Filter` element, you must also include the `Priority` and `DeleteMarkerReplication` elements\. In this example, `Priority` is irrelevant because there is only one rule\.  
In the following configuration, the filter specifies one prefix and two tags\. The rule applies to the subset of objects that have the specified key prefix and tags\. Specifically, it applies to object that have the `Tax/` prefix in their key names and the two specified object tags\. `Priority` doesn't apply because there is only one rule\.  

```
<?xml version="1.0" encoding="UTF-8"?>
<ReplicationConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <Role>arn:aws:iam::account-id:role/role-name</Role>
  <Rule>
    <Status>Enabled</Status>
    <Priority>1</Priority>
    <DeleteMarkerReplication>
       <Status>string</Status>
    </DeleteMarkerReplication>

    <Filter>
        <And>
          <Prefix>Tax/</Prefix>
          <Tag>
             <Tag>
                <Key>tagA</Key>
                <Value>valueA</Value>
             </Tag>
          </Tag>
          <Tag>
             <Tag>
                <Key>tagB</Key>
                <Value>valueB</Value>
             </Tag>
          </Tag>
       </And>

    </Filter>

    <Destination><Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET</Bucket></Destination>

  </Rule>
</ReplicationConfiguration>
```
You can specify a storage class for the object replicas as follows\.  

```
<?xml version="1.0" encoding="UTF-8"?>

<ReplicationConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <Role>arn:aws:iam::account-id:role/role-name</Role>
  <Rule>
    <Status>Enabled</Status>
    <Destination>
       <Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET</Bucket>
       <StorageClass>storage-class</StorageClass>
    </Destination>
  </Rule>
</ReplicationConfiguration>
```
You can specify any storage class that Amazon S3 supports\.

**Example 2: Replication configuration with two rules**  

**Example**  
In the following replication configuration:  
+ Each rule filters on a different key prefix so that each rule applies to a distinct subset of objects\. In this example, Amazon S3 replicates objects with the key names *`Tax/doc1.pdf`* and *`Project/project1.txt`*, but it doesn't replicate objects with the key name *`PersonalDoc/documentA`*\. 
+ Rule priority is irrelevant because the rules apply to two distinct sets of objects\. The next example shows what happens when rule priority is applied\. 
+ The second rule specifies the S3 Standard\-IA storage class for object replicas\. Amazon S3 uses the specified storage class for those object replicas\.
   

```
<?xml version="1.0" encoding="UTF-8"?>

<ReplicationConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <Role>arn:aws:iam::account-id:role/role-name</Role>
  <Rule>
    <Status>Enabled</Status>
    <Priority>1</Priority>
    <DeleteMarkerReplication>
       <Status>string</Status>
    </DeleteMarkerReplication>
    <Filter>
        <Prefix>Tax</Prefix>
    </Filter>
    <Status>Enabled</Status>
    <Destination>
      <Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET1</Bucket>
    </Destination>
     ...
  </Rule>
 <Rule>
    <Status>Enabled</Status>
    <Priority>2</Priority>
    <DeleteMarkerReplication>
       <Status>string</Status>
    </DeleteMarkerReplication>
    <Filter>
        <Prefix>Project</Prefix>
    </Filter>
    <Status>Enabled</Status>
    <Destination>
      <Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET1</Bucket>
     <StorageClass>STANDARD_IA</StorageClass>
    </Destination>
     ...
  </Rule>


</ReplicationConfiguration>
```

**Example 3: Replication configuration with two rules with overlapping prefixes**  <a name="overlap-rule-example"></a>
In this configuration, the two rules specify filters with overlapping key prefixes, *`star/`* and *`starship`*\. Both rules apply to objects with the key name *`starship-x`*\. In this case, Amazon S3 uses the rule priority to determine which rule to apply\. The higher the number, the higher the priority\.  

```
<ReplicationConfiguration>

  <Role>arn:aws:iam::account-id:role/role-name</Role>

  <Rule>
    <Status>Enabled</Status>
    <Priority>1</Priority>
    <DeleteMarkerReplication>
       <Status>string</Status>
    </DeleteMarkerReplication>
    <Filter>
        <Prefix>star</Prefix>
    </Filter>
    <Destination>
      <Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET1</Bucket>
    </Destination>
  </Rule>
  <Rule>
    <Status>Enabled</Status>
    <Priority>2</Priority>
    <DeleteMarkerReplication>
       <Status>string</Status>
    </DeleteMarkerReplication>
    <Filter>
        <Prefix>starship</Prefix>
    </Filter>    
    <Destination>
      <Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET1</Bucket>
    </Destination>
  </Rule>
</ReplicationConfiguration>
```

**Example 4: Example walkthroughs**  
For example walkthroughs, see [Walkthroughs: Examples for configuring replication](replication-example-walkthroughs.md)\.

For more information about the XML structure of replication configuration, see [PutBucketReplication](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTBucketPUTreplication.html) in the *Amazon Simple Storage Service API Reference*\. 

## Backward compatibility<a name="replication-backward-compat-considerations"></a>

The latest version of the replication configuration XML is V2\. XML V2 replication configurations are those that contain the `Filter` element for rules, and rules that specify S3 Replication Time Control \(S3 RTC\)\.

To see your replication configuration version, you can use the `GetBucketReplication` API operation\. For more information, see [GetBucketReplication](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetBucketReplication.html) in the *Amazon Simple Storage Service API Reference*\. 

For backward compatibility, Amazon S3 continues to support the XML V1 replication configuration\. If you've used XML V1 replication configuration, consider the following issues that affect backward compatibility:
+ Replication configuration XML V2 includes the `Filter` element for rules\. With the `Filter` element, you can specify object filters based on the object key prefix, tags, or both to scope the objects that the rule applies to\. Replication configuration XML V1 supports filtering based only on the key prefix\. In that case, you add the `Prefix` directly as a child element of the `Rule` element, as in the following example\.

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <ReplicationConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
    <Role>arn:aws:iam::account-id:role/role-name</Role>
    <Rule>
      <Status>Enabled</Status>
      <Prefix>key-prefix</Prefix>
      <Destination><Bucket>arn:aws:s3:::DOC-EXAMPLE-BUCKET</Bucket></Destination>
  
    </Rule>
  </ReplicationConfiguration>
  ```

  For backward compatibility, Amazon S3 continues to support the V1 configuration\. 
+ When you delete an object from your source bucket without specifying an object version ID, Amazon S3 adds a delete marker\. If you use V1 of the replication configuration XML, Amazon S3 replicates delete markers that result from user actions\. In other words, Amazon S3 replicates the delete marker only if a user deletes an object\. If an expired object is removed by Amazon S3 \(as part of a lifecycle action\), Amazon S3 does not replicate the delete marker\. 

  In V2 replication configurations, you can enable delete marker replication for tag\-based rules\. For more information, see [Replicating delete markers between buckets](delete-marker-replication.md)\. 

 