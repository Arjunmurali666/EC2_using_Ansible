# EC2_using_Ansible
## This is an ansible playbook to create an AWS ec2 instance. You need to configure the AWS ID & key and install python boto3 before running the playbook. 

```ansible
---
  - name: Provision an EC2 Instance
    hosts: localhost
    gather_facts: False

    vars:
      instance_type: t2.micro
      security_group: ansible_security
      image: ami-0d8f6eb4f641ef691
      keypair: ansible
      region: us-east-2
      count: 1

    tasks:
      - name: "Create a security group"
        local_action:
          module: ec2_group
          name: "{{ security_group }}"
          description: Security Group
          region: "{{ region }}"
          rules:
            - proto: tcp
              from_port: 22
              to_port: 22
              cidr_ip: 0.0.0.0/0
            - proto: tcp
              from_port: 80
              to_port: 80
              cidr_ip: 0.0.0.0/0
          rules_egress:
            - proto: all
              cidr_ip: 0.0.0.0/0
        register: basic_firewall

      - name: "Launch new EC2"
        local_action: ec2
                      group={{ security_group }}
                      instance_type={{ instance_type}}
                      image={{ image }}
                      wait=true
                      region={{ region }}
                      keypair={{ keypair }}
                      count={{count}}
        register: ec2

      - name: "Wait for EC2 Instance to come online"
        local_action: wait_for
                      host={{ item.public_ip }}
                      port=22
                      state=started
        with_items: "{{ ec2.instances }}"

```

## Now you can check your AWS account to find the new EC2.
