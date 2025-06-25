template = '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: terraform
  name: terraform
spec:
  containers:
  - command:
    - sleep
    - "3600"
    image: hashicorp/terraform
    name: terraform
'''

tfvars = """
region = "${params.region}"
ami_id = "${params.ami_id}"
instance_type = "${params.type}"
"""


properties([
    parameters([
        choice(choices: ['apply', 'destroy'], description: 'Select your choice', name: 'action'),
        choice(choices: ['us-east-1', 'us-east-2', 'us-west-1', 'us-west-2'], description: 'Select region', name: 'region'),
        string(description: 'Enter AMI ID', name: 'ami_id', trim: true),
        string(description: 'Enter Instance type', name: 'type', trim: true)
        ])
    ])


podTemplate(cloud: 'kubernetes', label: 'terraform', yaml: template) {
node ("terraform") {
    container ("terraform") {
    stage ("Checkout SCM") {
        git branch: 'main', url: 'https://github.com/buginets/actions-terraform.git'
    }

withCredentials([usernamePassword(credentialsId: 'aws-creds', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
    stage ("Terraform Init") {
        sh "terraform init -backend-config=key=${params.region}/terraform.tfvars"
    }
    stage ("Terraform Plan") {
        writeFile file: 'kaizen.tfvars', text: tfvars
        sh "terraform plan -var-file kaizen.tfvars"
    }
if (params.action == "apply") {
    
    stage ("Terraform Apply") {
        writeFile file: 'kaizen.tfvars', text: tfvars    
        sh "terraform apply -var-file kaizen.tfvars -auto-approve"
    }
}
else {
    stage ("Terraform Destroy") {
        writeFile file: 'kaizen.tfvars', text: tfvars
        sh "terraform destroy -var-file kaizen.tfvars -auto-approve"
    }
}
}
}
}
}
