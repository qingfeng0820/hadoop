## Block file
* Block file name: blk_<block_id>
* Checksum file name: blk_<block_id>_<GENERATION_STAMP>.meta


## Checksum file content: 
* Version (short) + Checksum Type (byte) + bytesPerChecksum (int) + checksum + checksum ... + checksum
checksum: size is checksumSize bytes. A checksum is calculated for each chunk((bytesPerChecksum bytes).

* Checksum Type ---> DataChecksum

## DataChecksum:
    Type: has id (Checksum Type) and size(checksumSize).
    bytesPerChecksum (int)
    Checksum instance
### Checksum Types:
    CHECKSUM_NULL: Type -> 0, checksumSize -> 0
    CHECKSUM_CRC32: Type -> 1, checksumSize -> 4
    CHECKSUM_CRC32C: Type -> 2, checksumSize -> 4
    CHECKSUM_DEFAULT: Type -> 3, checksumSize -> 0 // This cannot be used to create DataChecksum
    CHECKSUM_MIXED: Type -> 4, checksumSize -> 0

## Chunk
### a chunk = bytesPerChecksum  +  checksumSize

## Packet
### Packet size = Header Size (PLEN+HLEN+HEADER) + chunk size * N

## DataXceiverServer
* Running on datanode, to accept socket connection from clients for IO
## DataXceiver
* When accepted a socket connection, new a DataXceiver to handle IO requests
## DFClient
* Communication with namenode and datanode

## DFSInputStream
* Choose datanode to read block by BlockReader ans retry other datanodes if failed

## BlockReader
* work on client, to receive the data from BlockSender.
* Connect the chosen datanode
* Call DataXceiver.readBlock to read data

## BlockSender
* work on datanode, and sent block data to client
###send block data format:
* Send checksumHeader
* Send packets
* ![](./img/send_block_data.png)
### In a packet
    // Each packet looks like:
    //   PLEN    HLEN      HEADER     CHECKSUMS  DATA
    //   32-bit  16-bit   <protobuf>  <variable length>
    //
    // PLEN:      Payload length
    //            = length(PLEN) + length(CHECKSUMS) + length(DATA)
    //            This length includes its own encoded length in
    //            the sum for historical reasons.
    //
    // HLEN:      Header length
    //            = length(HEADER)
    //
    // HEADER:    the actual packet header fields, encoded in protobuf
    // CHECKSUMS: the crcs for the data chunk. May be missing if
    //            checksums were not requested
    // DATA       the actual block data
* HeaderLength is the length of HEADER, which occupies 2 bytes.
* PacketLen = total length of chunks + total length of checksum + 4 -> packetLen is int, which occupies 4 bytes.
### Send packet
#### Normal transfer  (Packet size <= 4K)
* check sum before send if verifyChecksum set to true (if check sum not needed, use DataChecksum.Type.NULL)
* send PacketHeader to client
* send checksums (if sendChecksum is true) and chunks to client 
#### TransferTo (Packet size <= 64K)
* send PacketHeader to client
* send checksums to client (if sendChecksum is true)
* Use transferTo to send chunks to client
### An empty packet is sent to mark the end and read completion.

## FSOutputSummer 
### work on client, to send data to datanode
* buffer size [bytesPerChecksum * 9], checksum buffer size [checksumSize * 9]
* if the bytes array for writing is longer than buffer, write data directly from bytes array (write length: buffer.length) to DataPacket
* Otherwise, copy the data from the bytes array to buffer, then write data from buffer to DataPacket when buffer is full
* before writing to DataPacket, calculate checksum data to checksum buffer for each chunk based on the data in buffer
### write to DataPacket one chunk by one chunk (so if the buffer is full, there should be 9 chunks)
* write checksum data of the chunk from checksum buffer to DataPacket's buffer at checksum data's position
* write data of the chunk from bytes array or buffer to DataPacket's buffer at data's position
## DataPacket
* the buffer size (Packet size <= 64k) 
* When the buffer is full, put this DataPacket to DataStreamer's queue for sending to datanode
* Each DataPacket has a sequence number (>= 0) of the block
* Heart beat DataPacket's sequence number is -1
## DataStreamer
* It's a thread
* Count the sequence number for DataPackets for each block
* Before the first DataPacket, call namenode to add a block
* Get connection with chosen downstream datanode 
* Call DataXceiver.writeBlock to send the checksum of this block and pipeline datanodes to it
* Send block packets to datanode one by one
### send packet
* Send PacketHeader to downstream datanode of the pipeline 
* send checksums and chunks to downstream datanode of the pipeline 

## DataStreamer.ResponseProcessor
* It's a thread
* Wait for the ack of each DataPacket orderly by seqno

## BlockReceiver
* work on datanode to receive block data from client
* It has a PacketReceiver to receive the packets of the block

## BlockReceiver.PacketResponder
* Wait ack from the downstream in pipeline, then sent ack to upstream/client for each packet






