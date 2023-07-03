# Summary

This repository illustrates how to set up and use Event Driven Ansible (EDA) both from the command line as well as with the EDA Controller that is now part of Ansible Automation Platform 2.4.  In this example, a webhook event with a specified payload will run the playbook `say-hello.yml`.

Reference: https://www.ansible.com/blog/getting-started-with-event-driven-ansible

# To run this on the command line

`ansible-rulebook --rulebook rulebooks/webhook-example.yml -i inventory.yml --print-events`

To test, run the following scenarios in another terminal:

## This will execute the playbook

`curl -H 'Content-Type: application/json' -d "{\"message\": \"Ansible is super cool\"}" 127.0.0.1:5001/endpoint`

## This will execute the playbook and print the sender

`curl -H 'Content-Type: application/json' -d "{\"message\": \"Ansible is super cool\", \"sender\": \"Sammy\"}" 127.0.0.1:5001/endpoint`

## This will not execute the playbook

`curl -H 'Content-Type: application/json' -d "{\"message\": \"Ansible is super cool dude\"}" 127.0.0.1:5001/endpoint`

# To run this in Ansible Automation Platform (AAP) with the Event Driven Ansible (EDA) Controller

## In the AAP controller

### Define a Project

```
Name: EDA
Organizaton: Default
Source Control Type: Git
Source Control URL: https://github.com/t-prinz/ansible-eda.git
```

### Define an Inventory using the Project as the inventory source

Navigate to Resources -> Inventories -> Add -> Add inventory

```
Name: EDA Inventory
Organization: Default
```

Click Save and navigate to the Sources tab and click Add to add the Project as the inventory source

```
Name: EDA Project Inventory
Source: Sourced from a Project
Project: EDA
Inventory file: / (project root)
```

### Define a Job Template

Note that you need to enable "Prompt on launch" in the Variables section in order for the playbook to have access to `ansible_eda`

```
Name: EDA - Hello
Inventory: EDA Inventory
Project: EDA
Playbook: say-hello.yml
In the Variables section, enable "Prompt on launch"
```

### Define an Application

Navigate to Administration -> Applications and Add a new Application using the following parameters:

```
Name: EDA Application
Organization: Default
Authorization grant type: Resource owner password-based
Client type: Public
```

Click Save

It doesn't appear the you need the Client ID, but it may be good to keep for future reference.

### Define an access Token

Select Access -> Users 
Select the admin user and then the Tokens tab
Click Add to add a new token with the following parameters:

```
Application:  EDA Application (click the search icon and the Application previously defined should be listed)
Scope: Write
```

Click Save

Make note of the Token.  It may be good to keep the Refresh Token for future reference.

## In the EDA controller

### Define a Project

```
Name: EDA
Organizaton: Default
Source Control Type: Git
Source Control URL: https://github.com/t-prinz/ansible-eda.git
```

### Define the access Token

Select User Access -> Users
Select the admin user and then the Controller Tokens tab
Click Create controller token to add a new token using the following parameters:

```
Name: AAP Controller Token
Token: <use the Token previously created on the AAP controller>
```

Click Create controller token

### Create the Rulebook Activation

Select Views -> Rulebook Activations
Click Create rulebook activation to create an activation using the following parameters:

```
Name: Webhook
Project: EDA
Rulebook: webhook-example-aap.yml
Decision environment: Default Decision Environment
Restart policy: Always
```

Click Create rulebook activation

Select the History tab and then the Running activation that was just created

To test, run the following in a terminal

`curl -H 'Content-Type: application/json' -d "{\"message\": \"Ansible is super cool\"}" edactlr.example.com:8080/endpoint`
