# Terraform-Lock and Statefile
Configure Remote Backend in Terraform & DynamoDB Table for State Locking.

**Terraform State File**

Terraform is an Infrastructure as Code (IaC) tool used to define and provision infrastructure resources. The Terraform state file is a crucial component of Terraform that helps it keep track of the resources it manages and their current state. This file, often named `terraform.tfstate`, is a JSON or HCL (HashiCorp Configuration Language) formatted file that contains important information about the infrastructure's current state, such as resource attributes, dependencies, and metadata.

**Advantages of Terraform State File:**

1. **Resource Tracking**: The state file keeps track of all the resources managed by Terraform, including their attributes and dependencies. This ensures that Terraform can accurately update or destroy resources when necessary.

2. **Concurrency Control**: Terraform uses the state file to lock resources, preventing multiple users or processes from modifying the same resource simultaneously. This helps avoid conflicts and ensures data consistency.

3. **Plan Calculation**: Terraform uses the state file to calculate and display the difference between the desired configuration (defined in your Terraform code) and the current infrastructure state. This helps you understand what changes Terraform will make before applying them.

4. **Resource Metadata**: The state file stores metadata about each resource, such as unique identifiers, which is crucial for managing resources and understanding their relationships.

**Disadvantages of Storing Terraform State in Version Control Systems (VCS):**

1. **Security Risks**: Sensitive information, such as API keys or passwords, may be stored in the state file if it's committed to a VCS. This poses a security risk because VCS repositories are often shared among team members.

2. **Versioning Complexity**: Managing state files in VCS can lead to complex versioning issues, especially when multiple team members are working on the same infrastructure.

## Creating EC2 instance, S3 bucket for statefile storing and DynamoDB for statefile locking:

Excecuting main.tf and initializing terraform project
```
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "Ubaid" {
  instance_type = "t2.micro"
  ami = "ami-0ecb62995f68bb549" # change this
  subnet_id = " subnet-05aff75cc18978861" # change this
}

resource "aws_s3_bucket" "s3_bucket" {
  bucket = "Ubaid-s3-demo-xyz" # change this
}

resource "aws_dynamodb_table" "terraform_lock" {
  name           = "terraform-lock"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```


<img width="701" height="265" alt="image" src="https://github.com/user-attachments/assets/df70b237-06e6-417c-9607-de9ad27fc5ef" />

<img width="1328" height="568" alt="image" src="https://github.com/user-attachments/assets/26f31c11-7a20-441c-8fac-2bcd0add9567" />

<img width="796" height="395" alt="image" src="https://github.com/user-attachments/assets/df1c387d-cd5d-4591-b088-f9d41ea9eff9" />


<img width="1278" height="197" alt="image" src="https://github.com/user-attachments/assets/1bae833b-a74f-498f-bacc-95c0d8e0688d" />

<img width="1023" height="283" alt="image" src="https://github.com/user-attachments/assets/91a64154-ed47-473a-b435-91437641d9dc" />


<img width="1283" height="268" alt="image" src="https://github.com/user-attachments/assets/b7612287-a02e-434b-afb9-9d96a81f99d3" />


<img width="1457" height="262" alt="image" src="https://github.com/user-attachments/assets/e103ce29-3c1f-4ee1-81ec-af826a4a326c" />

**We can see terraform state file exists in the local workspace, this project is to move the state file into s3 bucket, because it conatains sensitive information**

<img width="1063" height="553" alt="image" src="https://github.com/user-attachments/assets/49e05210-59e2-4a4b-9f8f-e4b9e0b15215" />

**Now initializing project with backend.tf and we can see statefile is stored in S3 bucket instead of storing in local workspace**

<img width="987" height="301" alt="image" src="https://github.com/user-attachments/assets/6189c7a7-dedc-4a71-8d30-a193f366d505" />

<img width="1671" height="439" alt="image" src="https://github.com/user-attachments/assets/c8ea7a46-a767-42a2-9723-d66980c1d18f" />




**Overcoming Disadvantages with Remote Backends (e.g., S3):**

A remote backend stores the Terraform state file outside of your local file system and version control. Using S3 as a remote backend is a popular choice due to its reliability and scalability. Here's how to set it up:

1. **Create an S3 Bucket**: Create an S3 bucket in your AWS account to store the Terraform state. Ensure that the appropriate IAM permissions are set up.

2. **Configure Remote Backend in Terraform:**

   ```hcl
   # In your Terraform configuration file (e.g., main.tf), define the remote backend.
   terraform {
     backend "s3" {
       bucket         = "your-terraform-state-bucket"
       key            = "path/to/your/terraform.tfstate"
       region         = "us-east-1" # Change to your desired region
       encrypt        = true
       dynamodb_table = "your-dynamodb-table"
     }
   }
   ```

   Replace `"your-terraform-state-bucket"` and `"path/to/your/terraform.tfstate"` with your S3 bucket and desired state file path.

3. **DynamoDB Table for State Locking:**

   To enable state locking, create a DynamoDB table and provide its name in the `dynamodb_table` field. This prevents concurrent access issues when multiple users or processes run Terraform.

**State Locking with DynamoDB:**

DynamoDB is used for state locking when a remote backend is configured. It ensures that only one user or process can modify the Terraform state at a time. Here's how to create a DynamoDB table and configure it for state locking:

1. **Create a DynamoDB Table:**

   You can create a DynamoDB table using the AWS Management Console or AWS CLI. Here's an AWS CLI example:

   ```sh
   aws dynamodb create-table --table-name your-dynamodb-table --attribute-definitions AttributeName=LockID,AttributeType=S --key-schema AttributeName=LockID,KeyType=HASH --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
   ```

2. **Configure the DynamoDB Table in Terraform Backend Configuration:**

   In your Terraform configuration, as shown above, provide the DynamoDB table name in the `dynamodb_table` field under the backend configuration.

By following these steps, you can securely store your Terraform state in S3 with state locking using DynamoDB, mitigating the disadvantages of storing sensitive information in version control systems and ensuring safe concurrent access to your infrastructure. For a complete example in Markdown format, you can refer to the provided example below:

```markdown
# Terraform Remote Backend Configuration with S3 and DynamoDB

## Create an S3 Bucket for Terraform State

1. Log in to your AWS account.

2. Go to the AWS S3 service.

3. Click the "Create bucket" button.

4. Choose a unique name for your bucket (e.g., `your-terraform-state-bucket`).

5. Follow the prompts to configure your bucket. Ensure that the appropriate permissions are set.

## Configure Terraform Remote Backend

1. In your Terraform configuration file (e.g., `main.tf`), define the remote backend:

   ```hcl
   terraform {
     backend "s3" {
       bucket         = "your-terraform-state-bucket"
       key            = "path/to/your/terraform.tfstate"
       region         = "us-east-1" # Change to your desired region
       encrypt        = true
       dynamodb_table = "your-dynamodb-table"
     }
   }
   ```

   Replace `"your-terraform-state-bucket"` and `"path/to/your/terraform.tfstate"` with your S3 bucket and desired state file path.

2. Create a DynamoDB Table for State Locking:

   ```sh
   aws dynamodb create-table --table-name your-dynamodb-table --attribute-definitions AttributeName=LockID,AttributeType=S --key-schema AttributeName=LockID,KeyType=HASH --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5
   ```

   Replace `"your-dynamodb-table"` with the desired DynamoDB table name.

3. Configure the DynamoDB table name in your Terraform backend configuration, as shown in step 1.

By following these steps, you can securely store your Terraform state in S3 with state locking using DynamoDB, mitigating the disadvantages of storing sensitive information in version control systems and ensuring safe concurrent access to your infrastructure.
```

Please note that you should adapt the configuration and commands to your specific AWS environment and requirements.
