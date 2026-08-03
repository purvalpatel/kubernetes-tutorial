Description:
-----------
### Stateful:
This  kubernetes controller is used when:
- Store data between sessions - they maintain context or state.
1. You need stable, unique network identifiers.
2. You want persistent storage
3. You need ordered, graceful deployment and scaling.

Commonly used for,
- Databases
- Zookeeper
- Apache point
- Kafka
- Elastic search
<br>
In kubernetes use statefulset instead of deployment.
<br>

### Stateless:

This is used when,<br>
- Do not maintain any information about previous interactions.
- Each request is independent.

Eg. <br>
1. Frontend
2. REST API
3. Nginx
4. Apache

Advantages: <br>
- Easy to scale
- Failover and recovery are simple

- Each pod has stable host name
- Get Unique PersistentVolumeChain.
- Is deployed and scaled in order.
