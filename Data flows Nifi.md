# Data flows

- Manage flow of information between systems :
  - routing/filtering
  - parsing
  - transforming
- not targeting latency but throughput + delivery guaranties

This comes before the datalake.

## Apache NiFi

- Flow file : Any incoming data is considered as a file with content, attributes, provenance.
- Processors
- Back pressure mechanism : if a queue is full, it will redirect the flow to another queue so that nothing is lost
