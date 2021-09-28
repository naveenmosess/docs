# PLDM transport wrapper on OpenBMC

  Velumani T(velu),  [velumanit@hcl](mailto:velumanit@hcl.com)
  Naveen Moses S(naveen.moses), [naveen.mosess@hcl.com](mailto:naveen.mosess@hcl.com)

Primary assignee: Naveen Moses S

Created: September 27, 2021

## Problem Description
On OpenBMC, PLDM stack currently supports pldm over MCTP transport layer.
There are some platforms which support other transport layers
ex : PLDM over NCSI based on RBT.

This design proposes a simple wrapper which detects and configures the
NIC supported transoprt layer so that PLDM request calls from
other processes are done via abstracted common pldm send/receive
request and the wrapper class routes the respective pldm api
call based on the detected configuration.

## Background and References

pldm repo link :
https://github.com/openbmc/pldm

pldm stack design :
https://github.com/openbmc/docs/blob/master/designs/pldm-stack.md

## Requirements

-The NIC supported configurations should be part of the BMC inventory.

-The wrapper  should have a detect method which reads the NIC capability from
the inventory or entity manager config.

- the wrapper  should have common method for send / receive PLDM requests.

- the dependent transport layer libraries should be included in the PLDM repository.

- initialization of the transport layer and drivers should be handled separatively and not
part of this wrapper.

#### Proposed wrapper design

A wrapper class library with detection method which reads the NIC card
supported transport layer.

example flow:

 - The library is linked with the supported transport layer libraries

- The wrapper class has common api for sending and receiving pldm requests

- Based on the initial detected configuration the corresponding transport layer
pldm request api is called(MCTP or NCSI).


##### Proposed requester design - flow diagrams

a) NCSI wrapper blocking API

```
+---------------+               +----------------+            +----------------+               +-----------------+
|BMC requester/ |               |PLDM wrapper    |            | NCSI layer     |               |MCTP layer       |
|client app     |               |lib (part of    |            |                |               |                 |
|               |               |libpldm)        |            |                |               |                 |
+-------+-------+               +-------+--------+            +--------+-------+               +---------+-------+
        |                               |                              |                                 |
        |App starts                     |                              |                                 |
        |                               |                              |                                 |
        +------------------------------->identify and config NIC       |                                 |
        |detect_nic()                   |transport layer supported     |                                 |
        |                               |                              |                                 |
        +<-------+return_code+----------+                              |                                 |
        |                               |                              |                                 |
        |                               |                              |                                 |
        |                               |                              |                                 |
        +------------------------------>+                              |                                 |
        |send_recv(cmd code, args)|     |                              |                                 |
        |                               |          send_command()      |                                 |
        |                               +----------------------------->+                                 |
        |                               |                              |                                 |
        |                               |                              |                                 |
        |                               |                              |                                 |
        |                               +<----+ ncssi response+--------+                                 |
        |                               |                              |                                 |
        |                               |                              |                                 |
        +<----+returns pldm_msg+--------+                              |                                 |
        |                               |                              |                                 |
        |                               |                              |                                 |
        +
```

##### Alternative to the proposed requester design

1. use the pldm stacks pldmd service to send / receive
pldm messages from other apps.

2. The requester part of the pldmd code should be extended to support
the wrapper logic to detect and call the respective transport layer library
api calls.


### Detection of NIC supported transport layer


## Impacts
The applications which are using mctp specific pldm request apis
should have to use the wrapper's common api.

## Testing
Testing can be done without having to depend on the underlying transport layer.

