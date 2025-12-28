# aws-systems-manager-distributor
Software Configuration Management and Automated Deployment


AWS Systems Manager: Centralized Software Distribution with Distributor
📖 Project Overview
Managing third-party software and custom agents across a fleet of servers is a common operational bottleneck. This project demonstrates how to use AWS Systems Manager Distributor to package software (the CloudWatch Agent) and Run Command to deploy it at scale.

The Scenario: As an administrator, you need a reliable way to ensure that 2,500+ EC2 instances across multiple regions have the latest version of a specific software package installed. Instead of logging into each server or writing complex shell scripts, I utilized AWS Systems Manager to create a repeatable, version-controlled distribution pipeline.



Technical Stack
AWS Systems Manager (Distributor): For creating and versioning custom software packages.

AWS Systems Manager (Run Command): For executing one-time or bulk software installations.

Amazon S3: Used as the centralized repository to store the software artifacts and manifest files.

Amazon EC2: Managed nodes that receive and install the packages.


🚀 Implementation Steps
1. Custom Package Creation
I created a Distributor package named cloudwatchagent using the "Simple" creation workflow:

Artifact Storage: Uploaded the software binary (RPM package) to a dedicated S3 bucket.

Metadata Mapping: Configured the Manifest to map the binary to specific target platforms (amazon linux) and architectures (x86_64).

Automated Scripting: Utilized Distributor’s auto-generated scripts to handle the installation via the yum package manager.



2. Fleet-Wide Deploymen
Once the package was "Active," I deployed it to multiple Test-Instance EC2 nodes:

Run Command Orchestration: Used the AWS-ConfigureAWSPackage document.

Targeting: Selected instances manually (though tag-based targeting was identified as the enterprise-scale best practice).

Validation: Monitored the command status in real-time until a "Success" state was reached across all nodes.


🔧 Troubleshooting & Key Insights
1. Package Installation Failures
Manifest Mismatch: If the installation fails with a "Platform not supported" error, double-check the manifest.json file. It must explicitly map the package to the correct OS (e.g., amazon) and Architecture (x86_64).

S3 Permissions: The EC2 instances require S3:Get permissions for the bucket where the Distributor package is stored. Without an appropriate IAM Instance Profile, the AWS-ConfigureAWSPackage document will fail to download the artifact.

2. Managed Node Connectivity
SSM Agent Status: Run Command will only work if the instance is a "Managed Node." If an instance doesn't appear in the target list, verify that the SSM Agent is running and that the instance has an outbound path to the Systems Manager service endpoints.

Ping Status: Always check that the instance shows as "Online" in the Systems Manager console before initiating a deployment.

3. Run Command Timeouts
In-Progress Locks: If you see an error like "Package is already in the process of other action," it usually means a previous installation attempt is still running or hung. Restarting the amazon-ssm-agent service on the instance typically clears this lock.

Output Truncation: If the Run Command output is cut off in the console, check the S3 Output (if enabled) or the local agent logs at /var/log/amazon/ssm/ for the full error trace.


graph LR
    A[SSM Automation] -->|1. Stop Instance| B(EC2 Instance)
    B -->|2. Modify Attribute| C{t3.small to t3.micro}
    C -->|3. Success| D[Start Instance]
    D -->|4. Final State| E(Running & Right-sized)
    
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style E fill:#00ff00,stroke:#333

    
Results
Centralization: Created a "Single Source of Truth" for software versions in the Distributor dashboard.

Scalability: Successfully deployed software to multiple nodes simultaneously without SSH access.

Auditability: Every installation was logged, providing a clear record of which version was installed on which instance and when.
