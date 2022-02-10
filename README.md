# Automating Deployment of Windows Servers 2022
The goal of this project is to deploy a ready-to-go windows server environment. This includes a domain controller, a replica domain controller, 
a DHCP server, and a fileserver. Additionally setting up users, groups, and OUs for the respective users within the domain.  <br>
To complete this project, 3 steps are taken. 
1. Use Packer to spin up a sys prepped and fully updated windows server 2022 iso for the environemnt
2. Use Terraform to deploy 4 virtual machines into a vSphere environment
3. Use Ansible to configure these 4 virtual machines as desired

## **Prerequisites**

  #### Linux machine w/ 
  - [Ansible](https://www.ansible.com/)
    - For Ubuntu:
        - `sudo apt update`
        - `sudo apt install software-properties-common`
        - `sudo add-apt-repository --yes --update ppa:ansible/ansible`
        - `sudo apt install ansible`
        - > Verify Ansible Install with `ansible`
  - [Terraform](https://www.terraform.io/)
    - `sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl`
    - `curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -`
    - `sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"`
    - `sudo apt-get update && sudo apt-get install terraform`
    - > Verify Terraform Insall with `terraform -help`
  - [Packer](https://www.packer.io/)
    - `curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -`
    - `sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"`
    - `sudo apt-get update && sudo apt-get install packer`
    - > Verify Packer Install with `packer`
  - [GitHub](https://git-scm.com/download/linux)
    - `sudo apt-get install git`

   #### Code Interpreter 
  - [Visual-Studio-Code](https://code.visualstudio.com/)
   #### vSphere Environment Setup
   - [vSphere](https://www.vmware.com/products/vsphere.html)
    - This project is using vSphere version 7.0.0
  <br>
## After all above criteria have been met
#### Ensure to Clone git repository to local linux machine
-   ```git clone git@github.com:Cinderblook/Terraform-Projects-NOCLOUD.git```
  
## Modify Variables in the following files to fit your environment
- Packer/configs/autounattend.xml
  - *Change Administrator Credentials*
- Packer/myvarfile.json
  - *Change to personal vSphere Data & Credentials*
- Packer/ws2022.pkr.hcl
  - *Cross check variables, alter datastore locations*
- Terraform/terraform.tfvars
  - *Use own custom Terraform variables, also vSphere Data & Credentials stored here*
- Ansible/group_vars/all.yaml
- Ansible/inventory.yml
- Ansible/winlab.yml
  <br>  
 ## Navigate to Packer Directory
- First setup Packer environment
    - `packer init -upgrade ws2022.pkr.hcl`
- Then apply the Packer configuration to create the Windows Server 2022 Image
    - `packer build -timestamp-ui -force -var-file=myvarfile.json ws2022.pkr.hcl`
* This packer execute pulls the newest windows server 2022 eval .iso from microsoft populates it into the vSphere environment, in the specified datacenter/cluster/host/datastore
* It then runs commands to: Grab DHCP, Updates the image, Enables SSH, Enables RDP, Configures necessary firewall settings, sets passwords/usernames, & installs VMware Tools to base image
* Additionally, it will install [Chocolatey](https://chocolatey.org/) for packages, notepad++, Edge, & 7-zip
  <br>
 ## After Packer Finishes
###### *Roughly an hour depending on processing and internet speed*
- Go into your vSphere and turn the resulting VM into a Template
  - Ensure this mimics the variables you have set in the terraform.tfvars file

 ## Navigate to Terraform Directory
- Setup Terraform Environemnt
  - `terraform init`
- Format terraform to ensure it meets criteria required
  - `terraform fmt`
- Do a terraform plan to detect any potential errors in code and to see potential end result. Read over this
  - `terraform plan`
- Finally, if all the above appears correct, perform a terraform apply
  - `terraform apply` ... `yes`
    - This may take awhile, once it is done, double check in vSphere all necessary Virtual Machines were created properly *(For me this took 20 minutes to fully complete)*

 ## Navigate to Ansible Directory
- Run your ansible playbook `ansible-playbook winlab.yml`
  - This should run through and detail each change 


# References Used/Useful Links
### I sourced various code and peices of information from the following Git Repositories
- Stefan Zimmermann [GitLab](https://gitlab.com/StefanZ8n/packer-ws2022) [Article](https://z8n.eu/2021/11/09/building-a-windows-server-2022-ova-with-packer/)
- Dmitry Teslya [GitHub](https://github.com/dteslya/win-iac-lab)
### Useful places for refernece
- Tutorials for Terraform, Packer, and others [Hashicorp-Tutorials](https://learn.hashicorp.com/search?query=Packer) 
- Ansible [Documentation](https://docs.ansible.com/)
  - [Windows-Modules](https://galaxy.ansible.com/ansible/windows?extIdCarryOver=true&sc_cid=701f2000001OH7YAAW)
- Terraform [Documentation](https://www.terraform.io/docs)
- Packer [Documentation](https://www.packer.io/docs)