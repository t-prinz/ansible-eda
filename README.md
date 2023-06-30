
Reference: https://www.ansible.com/blog/getting-started-with-event-driven-ansible

To run:

ansible-rulebook --rulebook rulebooks/webhook-example.yml -i inventory.yml --print-events

To test (in another terminal):

This will execute the playbook:

curl -H 'Content-Type: application/json' -d "{\"message\": \"Ansible is super cool\"}" 127.0.0.1:5001/endpoint

This will execute the playbook and print the sender:

curl -H 'Content-Type: application/json' -d "{\"message\": \"Ansible is super cool\", \"sender\": \"Trey\"}" 127.0.0.1:5001/endpoint

This will not execute the playbook:

curl -H 'Content-Type: application/json' -d "{\"message\": \"Ansible is super cool dude\"}" 127.0.0.1:5001/endpoint

To set up the AAP controler and EDA controller

In the AAP controller:

Select Administration -> Applications and Add a new Application using the following parameters:

Name: EDA Application
Organization: Default
Authorization grant type: Resource owner password-based
Client type: Public

Click Save

It doesn't appear the you need the Client ID, but it may be good to keep for future reference.

Select Access -> Users 
Select the admin user and then the Tokens tab
Click Add to add a new token with the following parameters:

Application:  EDA Application (click the search icon and the Application previously defined should be listed)
Scope: Write

Click Save

Make note of the Token.  It may be good to keep the Refresh Token for future reference.

In the AAP EDA controller:

Define your Project

Select User Access -> Users
Select the admin user and then the Controller Tokens tab
Click Create controller token to add a new token using the following parameters:

Name: AAP Controller Token
Token: <use the Token previously created on the AAP controller>

Click Create controller token

Select Views -> Rulebook Activations
Click Create rulebook activation to create an activation using the following parameters:

Name: Webhook
Project: EDA
Rulebook: webhook-example-aap.yml
Decision environment: Default Decision Environment
Restart policy: Always

Click Create rulebook activation

Select the History tab and then the Running activation that was just created

In a terminal, execute the following command:

curl -H 'Content-Type: application/json' -d "{\"message\": \"Ansible is super cool\"}" aws-aapeda.sandbox761.opentlc.com:8080/endpoint

In the AAP Controller, set up a Project, Credential, Inventory, and the Template to run when an event is received.  In this case, the Project uses this git repository and the Template is defined as follows:

Name: EDA - Hello
Inventory: <your inventory>
Project: EDA
Playbook: say-hello.yml
Credentials: <your credential>
