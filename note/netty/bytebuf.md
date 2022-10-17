# Byte buffer
## java.nio.ByteBuffer
* Hierarchy

  ![](./img/bytebuff_hierarchy.png)
  1. Direct Buffer: get direct buffer by allocateDirect which is not controlled by JVM
  2. Non-Direct Buffer: get non-direct buffer from JVM heap.
  3. HeapByteBufferR和DirectByteBufferR are read-only
* properties
  1. capacity：一个buffer的capacity是指它所包含的元素数量。该值永远不会为负数，也永远不会改变。（给定了初始值之后就不会改变了）。对于non-direct buffers，在内部维护一个字节数组用于存放元素，这里的capacity大小也就是指定了这个数组的大小
  2. limit：是buffer中第一个不能够被读取或写入的元素的索引值（元素是存放在数组中的，元素的索引值即数组的下标），该值永远不会为负数，并且不能比capacity值大。
  3. position：和limit相对，是buffer中第一个能够被读取或写入的元素的索引值。该值永远不会为负数，并且不能比limit值大。初始化的时候为0，每往buffer存放数据，position的值都会加1。
  4. mark：定义了这样的一个元素索引值，当调用reset方法时，position的值就等于mark值。若没有定义，调用reset方法会报InvalidMarkException。这个值从来不会被定义，但是一旦被定义，该值永远不会为负数，并且不能比position的值大。当定义了mark，如果调整之后的position或limit的值小于了mark值，则mark值会失效（值为-1）。mark未被定义时的值是-1，一旦被定义，值就不能是负数了。
  5. 0<=mark<=position<=limit<=capacity。一个新创建的buffer，position初始化为0，mark不会被定义，即值为-1。从limit、position的定义可以看出，最大的数据读取或写入长度是limit减去position。若position=0，limit=5，则此时最大的读取存放数据的长度就是5。数组的下标范围其实是从0到4。可以理解成是一种左闭右开的关系，即包括position的位置，不包括limit的位置。也可以说是读取存放数据的数组下标是从position开始到limit-1的下标位置。
  6. offset: 来确定元素的起始位置。默认值为0。可以认为是通过position和offset来确定元素的数组下标的。offset not used in direct bytebuffer (下图中typo： offect -> offset)
  7. address: The address (including offset) of allocated direct memory.

  ![](./img/bytebuffer_offset1.png)
  ![](./img/bytebuffer_offset2.png)
* operations
  1. read from bytebuffer: call clear or flip firstly to make position to 0
     clear sets limit to capacity, it can read all bytes in bytebuffer.
     flip sets limit to current position, it can only read bytes which were already consumed last time.
  2. write to bytebuffer: check remaining or calling compact of the bytebuffer before writing to it.
  3. mark and reset: make sets mark to current position. reset sets position to current mark.
  4. duplicate: duplicate a same bytebuffer which has it owner offset, mark, position, limit and capacity (keep them same as the original bytebuffer) but data (byte array) shared.
  5. slice: set offset = current position + current offset, limit = capacity = current remaining, position = 0 
  6. rewind: set position = 0, mark = -1 for repeat reading or overwriting.
  7. compact: discard consumed bytes by moving the bytes from current position to the start (offset) of the bytebuffer. Then set position = 0 and limit = capacity

  ![](./img/bytebuffer_compact.png)
  8. java.nio.ByteOrder: ByteOrder.nativeOrder(), buffer.order(ByteOrder.LITTLE_ENDIAN). Integer 和 Long 都提供了一个静态方法reverseBytes()
  
* sun.nio.ch.IOUtil
```
static int read(FileDescriptor var0, ByteBuffer var1, long var2, NativeDispatcher var4) throws IOException {
    if (var1.isReadOnly()) {
        throw new IllegalArgumentException("Read-only buffer");
    } else if (var1 instanceof DirectBuffer) {
        // If ByteBuffer is instance of DirectByteBuffer, read file content into this ByteBuffer
        return readIntoNativeBuffer(var0, var1, var2, var4);
    } else {
        // If ByteBuffer is instance of HeapByteBuffer, create a temporary DirectByteBuffer to get file content
        // Then copy the bytes from the temporary DirectByteBuffer to the HeapByteBuffer (one more copy then using DirectByteBuffer)
        // Because HeapByteBuffer is controlled by JVM, so it may be moved by "compacting GC".
        // But the ByteBuffer passed to the native code should not be moved during native code running.
        // So it cannot pass HeapByteBuffer to native code.
        ByteBuffer var5 = Util.getTemporaryDirectBuffer(var1.remaining());
        int var7;
        try {
            int var6 = readIntoNativeBuffer(var0, var5, var2, var4);
            var5.flip();
            if (var6 > 0) {
                var1.put(var5);
            }

            var7 = var6;
        } finally {
            Util.offerFirstTemporaryDirectBuffer(var5);
        }

        return var7;
    }
 }
```
* 在进行IO操作时，例如文件读写，或者socket读写，少了一次copy性能有提升。
  注意，仅限于有IO操作的场景下
* 在非IO操作的场景下，例如仅仅做数据的编解码，不和机器硬件打交道，那么即使使用了HeapByteBuffer也不会产生copy动作，如果此时仍然在使用DirectByteBuffer，由于数据存储部分在堆外存，内存空间的分配和释放比堆内存更加复杂一些，性能也稍慢一些，在netty中是通过buf池的方案来解决的
* new DirectByteBuffer时
  1. 会调用System.gc()去触发一个full gc (in Bits.reserveMemory)，当然前提是你没有显示的设置-XX:+DisableExplicitGC来禁用显式GC。但调用System.gc()并不能够保证full gc马上就能被执行。
  2. 之后会进行最多9次尝试，看JVM进程是否有足够的可用堆外内存来分配堆外内存。 （通过-XX:MaxDirectMemorySize来指定最大的堆外内存， 默认是67108864L）
  3. 每次尝试会触发一次非堵塞的Reference#tryHandlePending(false)。该方法会将已经被JVM垃圾回收的DirectBuffer对象的堆外内存释放。
  4. 如果9次尝试后还是没有足够的堆外内存，报堆外内存不足错误。

## io.netty.buffer.ByteBuf
*      +-------------------+------------------+------------------+
       | discardable bytes |  readable bytes  |  writable bytes  |
       |                   |     (CONTENT)    |                  |
       +-------------------+------------------+------------------+
       |                   |                  |                  |
       0      <=      readerIndex   <=   writerIndex    <=    capacity

# Pool
## io.netty.util.Recycler
* FastThreadLocal<Stack<T>> threadLocal: provide Stack to cache the instances of T
* protected abstract T newObject(Handle<T> handle): if no available instance of T in cache, new one.
* When cache an instance of T, Stack will create a DefaultHandle to wrap the instance of T.
* The DefaultHandle provides recycle method to re-cache the returned instance of T to the Stack.
