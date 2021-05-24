# DistributedWOL

DistributedWOL is a solution used to distribute a Wake Up On LAN (WOL) service accross several internal LANs.
The first version targets only the waking up of device. We certainly could increase the solution by adding the management of powering down the devices.

## Micro-Services Architecture

It is designed to run on a (or several) raspberry (for redundancy and network access) and it is composed by the following micro-services :
* DistributedWOL-UI : The service allows a user to control the WOL solution.
* DistributedWOL-API : The API exposes the entity and features of the solution. (Please note that each service exposes a low-level API, all API are compiled behind a Reverse Proxy)
    * Authentication : above JWT it allows a user (simple authentification) or a service to authenticate itself (APIKey)
    * Entity collections management (**C**reate/**R**ead/**U**pdate/**D**elete) : 
        * DistributedWOL-Gateway : represents a device able to send WOL message and retain state of the distributed solution.
        * DistributedWOL-Target : represents a device able to receive WOL message, each target can be accessible from several gateways but also inaccessible from some others.
        * DistributedWOL-User : represents a user able to connect to the distributed WOL Service.
        * DistributedWOL-App : represents an application to connect to the distributed WOL Service
        * DistributedWOL-Job : represents an asynchreous job that an authenticated user can create, cancel or monitor.
* DistributedWOL-Management : The service is responsible of any management operation :
    * Authentification management (JWT)
        * Authentication Process
        * Token Management
    * Gateway Management (CRUD)
    * Target Management (CRUD)
    * User/App Management (CRUD)
    * Job Management (CRUD)
* DistributedWOL-Worker : The service is responsible of any asynchroneous job on a device.
    * There are several job types :
        * Scanning the direct-attached network location for available services
        * Updating the conflict-free replicated data type (CRDT) database
        * Waking up a device
    * The service can work in three modes :
        * Polling mode : it regularly polls the management service to get newer job
        * Rising mode : through a call to its lower API, it can be alerted to poll a particular job from the management service
        * Scheduling mode : Doing the same job on a regular basis (each hour)

## CRDT goal
The goal to use a CRDT representation of the state is to be able to add a new device in the network and replicate the state of the solution on it.
Unfortunately, job aren't replicated but forwarded to the nearest gateway able to manage it.
The CRDT mechanism works based on the fact that a global solution App Key is shared between each device.
When a DistributedWOL-Gateway is added to an "origin" gateway, this last one call the new added device to check if the Solution App Key is functionnal.
Please note that for security purpose an HTTPS layer must be implemented.
A solution to provision certificate should be considered : EST Server/Client (https://tools.ietf.org/html/rfc7030)
