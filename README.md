# Project documentation

# On Premise Splunk Enterprise Deployment in AWS

![Python CI](https://github.com/dbaycoinc/splunkonpremiscloud/workflows/python.yml/badge.svg)
![Terraform CI](https://github.com/dbaycoinc/splunkonpremiscloud/workflows/terraform.yml/badge.svg)
![Coverage Status](https://img.shields.io/codecov/c/github/dbaycoinc/splunkonpremiscloud)

This repository provides a foundational GitOps setup for deploying and managing Splunk Enterprise in AWS using a combination of Terraform, Python, Bash, and YAML declarative configuration files. The workflow simplifies the provisioning, configuration, and scaling of Splunk indexers, search heads, deployers, and cluster masters.


### **Directory Structure**

```

.
├── Makefile
├── README.md
├── bash
│   ├── scripts
│   │   ├── common_functions.sh
│   │   ├── configure_splunk_node.sh
│   │   └── deploy_infrastructure.sh
│   └── tests
│       ├── test_configure_splunk_node.sh
│       └── test_deploy_infrastructure.sh
├── python
│   ├── main.py
│   ├── scripts
│   │   └── setup_splunk.sh
│   ├── splunk
│   │   ├── __init__.py
│   │   ├── deployer.py
│   │   ├── node_manager.py
│   │   ├── tests
│   │   │   ├── test_deployer.py
│   │   │   ├── test_node_manager.py
│   │   │   └── test_utils
│   │   │       ├── test_splunk_api.py
│   │   │       └── test_yaml_loader.py
│   │   └── utils
│   │       ├── __init__.py
│   │       ├── ec2_helper.py
│   │       ├── ec2_manager.py
│   │       ├── s3_backend.py
│   │       ├── splunk_api.py
│   │       └── yaml_loader.py
│   └── tests
│       ├── test_deployer.py
│       ├── test_node_manager.py
│       └── test_utils
│           ├── test_s3_backend.py
│           └── test_yaml_loader.py
├── requirements.txt
├── setup.py
├── splunk_deployment.yaml
└── terraform
    ├── envs
    │   ├── dev
    │   │   ├── backend.tf
    │   │   ├── main.tf
    │   │   └── variables.tfvars
    │   └── prod
    │       ├── backend.tf
    │       ├── main.tf
    │       └── variables.tfvars
    ├── modules
    │   ├── aws
    │   │   ├── nacl
    │   │   │   ├── main.tf
    │   │   │   ├── outputs.tf
    │   │   │   └── variables.tf
    │   │   └── vpc
    │   │       ├── main.tf
    │   │       ├── outputs.tf
    │   │       └── variables.tf
    │   └── splunk
    │       ├── main.tf
    │       ├── outputs.tf
    │       └── variables.tf
    └── vpc_config.yaml

```

---

## Prerequisites

1. **AWS Account and CLI**:
   - Ensure you have AWS CLI installed and configured with appropriate permissions.
   - Required permissions: `ec2:RunInstances`, `ec2:DescribeInstances`, `s3:CreateBucket`.

2. **Splunk Enterprise Installer**:
   - Ensure Splunk installation files are accessible (downloaded via the script or manually).

3. **Python Packages**:
   - Install the required Python packages:
     ```bash
     pip install boto3 pyyaml
     ```

4. **SSL Certificates**:
   - Provide the path to your SSL certificate for Splunk node configuration.

---

## Configuration

### `splunk_deployment.yaml`

Define your Splunk cluster infrastructure in this YAML file. Below is a sample configuration:

```yaml
splunk_cluster:
  name: "SplunkEnterpriseCluster"
  certificate: "/path/to/ssl/certificate.pem"
  s3_backend:
    bucket_name: "splunk-data-backup"
    region: "us-west-2"

nodes:
  indexers:
    count: 3
    instance_type: "m5.large"
    ami_id: "ami-1234567890abcdef"
    tags:
      Role: "SplunkIndexer"
  searchheads:
    count: 2
    instance_type: "t3.large"
    ami_id: "ami-0987654321abcdef"
    tags:
      Role: "SplunkSearchHead"
  deployer:
    count: 1
    instance_type: "t2.medium"
    ami_id: "ami-1111222233334444"
    tags:
      Role: "SplunkDeployer"
  cluster_master:
    count: 1
    instance_type: "t3.medium"
    ami_id: "ami-5555666677778888"
    tags:
      Role: "SplunkClusterMaster"

network:
  vpc_id: "vpc-abcdefgh"
  subnets:
    - "subnet-12345"
    - "subnet-67890"
````

* * *

Deployment Workflow
-------------------

### 1\. **Deploy Infrastructure**

Use the Python script `splunk_deployer.py` to launch and configure EC2 instances as defined in `splunk_deployment.yaml`:

```bash
python splunk_deployer.py
```

#### **Features of `splunk_deployer.py`**:

*   Provisions EC2 instances for Splunk roles (indexers, search heads, deployer, cluster master).
*   Configures S3 backend for Splunk archival.

* * *

### 2\. **Configure Nodes**

SSH into each instance and run the Bash script `configure_splunk_node.sh` with appropriate arguments:

```bash
./configure_splunk_node.sh <path_to_certificate> <node_role> <cluster_name> <s3_bucket>
```

#### **Example**:

```bash
./configure_splunk_node.sh /path/to/cert.pem indexer SplunkEnterpriseCluster splunk-data-backup
```

#### **Features of `configure_splunk_node.sh`**:

*   Installs Splunk Enterprise.
*   Configures the node for its respective role (indexer, search head, deployer, or cluster master).
*   Attaches S3 as the archival backend for indexers and cluster master.

* * *

Validation
----------

1.  **EC2 Instances**:
    
    *   Verify that all EC2 instances are running and properly tagged.
2.  **Splunk Configuration**:
    
    *   Check that Splunk nodes are configured with the correct roles:
        *   Indexers: Slave nodes in the cluster.
        *   Search Heads: Connected to the cluster master.
        *   Cluster Master: Manages the cluster configuration.
3.  **S3 Backend**:
    
    *   Ensure that the S3 bucket is created and accessible from the Splunk indexers and cluster master.

* * *

Additional Enhancements
-----------------------

1.  **Auto Scaling**:
    
    *   Integrate AWS Auto Scaling Groups for indexers and search heads.
2.  **Monitoring**:
    
    *   Use AWS CloudWatch for monitoring EC2 and S3 performance.
3.  **Automation**:
    
    *   Automate configuration using Ansible or Chef.

* * *

Contributing
------------

Feel free to submit issues or pull requests for new features and improvements. Contributions are welcome!

* * *

License
-------

This project is open-sourced under the MIT License.


---

This will serves as a comprehensive guide for deploying and managing Splunk Enterprise in AWS as it grows. This project and its configurations are entirely original and based on a hypothetical design approach for a mock-up company. It does not replicate, copy, or reuse any proprietary configurations or designs from any previous companies or projects. Instead, it reflects my personal approach to redesigning and managing Splunk Enterprise in AWS, based on industry best practices and my professional expertise. Let me know if you need further modifications or additional sections!
