---
date: "2020-11-19T19:04:54+05:30"
title: "keptn delete service"
slug: keptn_delete_service
---
## keptn delete service

Deletes a service from a project

### Synopsis

Deletes a service from a project by deleting the configuration in the GIT repository.
Furthermore, if Keptn is used for continuous delivery (i.e. services have been onboarded), this command will also uninstall the associated Helm releases.


```
keptn delete service SERVICENAME --project=PROJECTNAME [flags]
```

### Examples

```
keptn delete service carts --project=sockshop
```

### Options

```
  -h, --help             help for service
  -p, --project string   The project from which to delete the service
```

### Options inherited from parent commands

```
      --mock                 Disables communication to a Keptn endpoint
  -n, --namespace string     Specify the namespace where Keptn should be installed, used and uninstalled in (default keptn). (default "keptn")
  -q, --quiet                Suppresses debug and info messages
      --suppress-websocket   Disables WebSocket communication to suppress info messages from services running inside Keptn
  -v, --verbose              Enables verbose logging to print debug messages
```

### SEE ALSO

* [keptn delete](../keptn_delete/)	 - Deletes a project

###### Auto generated by spf13/cobra on 19-Nov-2020
