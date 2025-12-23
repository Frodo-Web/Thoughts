# SIGUSR1 and SIGUSR2
These are user-defined signals in POSIX-compliant operating systems, meaning its specific function is determined by the program receiving it, rather than having a default system action like termination or interruption.

Use cases:
- Log Rotation
- Configuration Reloading
- Debugging/Status Information (trigger dumps or reports)
- Synchronization (inter-process synchronization or control)

## SIGUSR1-2 as Health checks!
```
      livenessProbe:
        exec:
          command:
            - /bin/bash
            - '-c'
            - pkill -USR1 perl
        initialDelaySeconds: 120
        timeoutSeconds: 1
        periodSeconds: 60
        successThreshold: 1
        failureThreshold: 3

```
