## SERVICE
### Protocol
yarn_service_protos.proto
Define RequestProto and ResponseProto for services
Generated org.apache.hadoop.yarn.proto.YarnServiceProtos which includes RegisterApplicationMasterRequestProto, RegisterApplicationMasterResponseProto and so on

applicationmaster_protocol.proto
Define ApplicationMasterProtocolService
Generated org.apache.hadoop.yarn.proto.ApplicationMasterProtocol which includes org.apache.hadoop.yarn.proto.ApplicationMasterProtocolService

ApplicationMasterProtocolService.Interface
com.google.protobuf.Service ApplicationMasterProtocolService.newReflectiveService(Interface impl)
(ApplicationMasterProtocolService extends com.google.protobuf.Service itself)

ApplicationMasterProtocolService.Stub
Stub ApplicationMasterProtocolService.newStub(com.google.protobuf.RpcChannel channel)

ApplicationMasterProtocolService.BlockingInterface
com.google.protobuf.BlockingService ApplicationMasterProtocolService.newReflectiveBlockingService(BlockingInterface impl)

ApplicationMasterProtocolService.BlockingStub
BlockingInterface ApplicationMasterProtocolService.newBlockingStub(com.google.protobuf.BlockingRpcChannel channel)



### PB Service
Interface (Which is the protocol)
org.apache.hadoop.yarn.api.ApplicationMasterProtocolPB (extends ApplicationMasterProtocolService.BlockingInterface)

Implementation
org.apache.hadoop.yarn.api.impl.pb.service.ApplicationMasterProtocolPBServiceImpl
ApplicationMasterProtocolPBServiceImpl(org.apache.hadoop.yarn.api.ApplicationMasterProtocol realServiceImpl)

PB Service accept protocol buffer request, then convert this request to the request for the real service.
Then the real service returns the response which need to be converted to protocol buffer response by PB Service.



### Real Service
Interface
org.apache.hadoop.yarn.api.ApplicationMasterProtocol

Implementation
org.apache.hadoop.yarn.server.resourcemanager.ApplicationMasterService (Server side)
org.apache.hadoop.yarn.api.impl.pb.client.ApplicationMasterProtocolPBClientImpl (Client side)


HadoopYarnProtoRPC.getProxy HadoopYarnProtoRPC.getServer uses org.apache.hadoop.yarn.api.ApplicationMasterProtocol
Then RpcServerFactoryPBImpl uses org.apache.hadoop.yarn.api.ApplicationMasterProtocolPB to call RPC layer to deploy service
RpcClientFactoryPBImpl uses reflection to get instance of org.apache.hadoop.yarn.api.impl.pb.client.ApplicationMasterProtocolPBClientImpl
ApplicationMasterProtocolPBClientImpl uses org.apache.hadoop.yarn.api.ApplicationMasterProtocolPB to call the RPC service









