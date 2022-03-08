# Using the AWS SDK for Java to create an Amazon EMR cluster<a name="calling-emr-with-java-sdk"></a>

The AWS SDK for Java provides packages with Amazon EMR functionality:
+  [software\.amazon\.awssdk\.services\.emr](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/emr/package-summary.html)
+  [software\.amazon\.awssdk\.services\.emr\.model](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/emr/model/package-summary.html)
+  [software\.amazon\.awssdk\.services\.emr\.paginators](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/emr/paginators/package-summary.html)
+  [software\.amazon\.awssdk\.services\.emr\.transform](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/emr/transform/package-summary.html)
+  [software\.amazon\.awssdk\.services\.emr\.waiters](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/emr/waiters/package-summary.html)
+  [software\.amazon\.awssdk\.services\.emr\.waiters\.internal](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/emr/waiters/internal/package-summary.html)
+  [software\.amazon\.awssdk\.services\.emrcontainers](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/emrcontainers/package-summary.html)
+  [software\.amazon\.awssdk\.services\.emrcontainers\.model](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/emrcontainers/model/package-summary.html)
+  [software\.amazon\.awssdk\.services\.emrcontainers\.paginators](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/emrcontainers/paginators/package-summary.html)
+  [software\.amazon\.awssdk\.services\.emrcontainers\.transform](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/emrcontainers/transform/package-summary.html)

For more information about these packages, see the [AWS SDK for Java API Reference](https://docs.aws.amazon.com/sdk-for-java/latest/reference/)\.

The following example illustrates how the SDKs can simplify programming with Amazon EMR\. The code sample below uses the `StepFactory` object, a helper class for creating common Amazon EMR step types, to create an interactive Hive cluster with debugging enabled\. 

```
import com.amazonaws.AmazonClientException;
import com.amazonaws.auth.AWSCredentials;
import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.profile.ProfileCredentialsProvider;
import com.amazonaws.services.elasticmapreduce.AmazonElasticMapReduce;
import com.amazonaws.services.elasticmapreduce.AmazonElasticMapReduceClientBuilder;
import com.amazonaws.services.elasticmapreduce.model.*;
import com.amazonaws.services.elasticmapreduce.util.StepFactory;

public class Main {

	public static void main(String[] args) {
		AWSCredentialsProvider profile = null;		
		try {
			credentials_profile = new ProfileCredentialsProvider("default"); // specifies any named profile in .aws/credentials as the credentials provider
        } catch (Exception e) {
            throw new AmazonClientException(
                    "Cannot load credentials from .aws/credentials file. " +
                    "Make sure that the credentials file exists and that the profile name is defined within it.",
                    e);
        }
		
		// create an EMR client using the credentials and region specified in order to create the cluster
		AmazonElasticMapReduce emr = AmazonElasticMapReduceClientBuilder.standard()
			.withCredentials(credentials_profile)
			.withRegion(Regions.US_WEST_1)
			.build();
        
        // create a step to enable debugging in the AWS Management Console
		StepFactory stepFactory = new StepFactory(); 
		StepConfig enabledebugging = new StepConfig()
   			.withName("Enable debugging")
   			.withActionOnFailure("TERMINATE_JOB_FLOW")
   			.withHadoopJarStep(stepFactory.newEnableDebuggingStep());
        
        // specify applications to be installed and configured when EMR creates the cluster
		Application hive = new Application().withName("Hive");
		Application spark = new Application().withName("Spark");
		Application ganglia = new Application().withName("Ganglia");
		Application zeppelin = new Application().withName("Zeppelin");
		
		// create the cluster
		RunJobFlowRequest request = new RunJobFlowRequest()
	       		.withName("MyClusterCreatedFromJava")
	       		.withReleaseLabel("emr-5.20.0") // specifies the EMR release version label, we recommend the latest release
	       		.withSteps(enabledebugging)
	       		.withApplications(hive,spark,ganglia,zeppelin)
	       		.withLogUri("s3://path/to/my/emr/logs") // a URI in S3 for log files is required when debugging is enabled
	       		.withServiceRole("EMR_DefaultRole") // replace the default with a custom IAM service role if one is used
	       		.withJobFlowRole("EMR_EC2_DefaultRole") // replace the default with a custom EMR role for the EC2 instance profile if one is used
	       		.withInstances(new JobFlowInstancesConfig()
	       	   		.withEc2SubnetId("subnet-12ab34c56")
	           		.withEc2KeyName("myEc2Key") 
	           		.withInstanceCount(3) 
	           		.withKeepJobFlowAliveWhenNoSteps(true)    
	           		.withMasterInstanceType("m4.large")
	           		.withSlaveInstanceType("m4.large"));

	   RunJobFlowResult result = emr.runJobFlow(request);  
	   System.out.println("The cluster ID is " + result.toString());

	}

}
```

At minimum, you must pass a service role and jobflow role corresponding to EMR\_DefaultRole and EMR\_EC2\_DefaultRole, respectively\. You can do this by invoking this AWS CLI command for the same account\. First, look to see if the roles already exist: 

```
aws iam list-roles | grep EMR
```

Both the instance profile \(EMR\_EC2\_DefaultRole\) and the service role \(EMR\_DefaultRole\) will be displayed if they exist: 

```
"RoleName": "EMR_DefaultRole", 
            "Arn": "arn:aws:iam::AccountID:role/EMR_DefaultRole"
            "RoleName": "EMR_EC2_DefaultRole", 
            "Arn": "arn:aws:iam::AccountID:role/EMR_EC2_DefaultRole"
```

If the default roles do not exist, you can use the following command to create them:

```
aws emr create-default-roles
```
