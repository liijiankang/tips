# 剧情回顾
___
* 在create流程中返回了DFSOutputStream 对象，DFSOutputStream 对象继承了Daemon对象。
* 在newStreamForCreate方法返回DFSOutputStream 对象之前启动了DataStreamer守护进程

**代码分析**
```java
    static DFSOutputStream newStreamForCreate(DFSClient dfsClient, String src,
      FsPermission masked, EnumSet<CreateFlag> flag, boolean createParent,
      short replication, long blockSize, Progressable progress,
      DataChecksum checksum, String[] favoredNodes, String ecPolicyName)
      throws IOException {
      ...
      //纠删码与副本写入分流
      if(stat.getErasureCodingPolicy() != null) {
        out = new DFSStripedOutputStream(dfsClient, src, stat,
            flag, progress, checksum, favoredNodes);
      } else {
        out = new DFSOutputStream(dfsClient, src, stat,
            flag, progress, checksum, favoredNodes, true);
      }
      //调用DFSOutputStream对象的start方法
      out.start();
      return out;
    }
  }
```
**DFSOutputStream.start方法**
```java
    protected synchronized void start() {
    getStreamer().start();
  }
```
**DFSOutputStream.getStreamer**
```java
    protected DataStreamer getStreamer() {
    return streamer;
  }
```
* 可以看到在create流程中返回DFSOutputStream 对象的同时启动了DataStreamer.run进程
___
# write流程分析
## DataStreamer.run进程分析
```java
   @Override
  public void run() {
    long lastPacket = Time.monotonicNow();
    TraceScope scope = null;
    ...
      DFSPacket one;
      try {
        // process datanode IO errors if any
        boolean doSleep = processDatanodeOrExternalError();
        //dfs.client.socket-timeout
        //READ_TIMEOUT = 60 * 1000
        //WRITE_TIMEOUT = 8 * 60 * 100060 * 1000
        final int halfSocketTimeout = dfsClient.getConf().getSocketTimeout()/2;
        synchronized (dataQueue) {
          // wait for a packet to be sent.
          long now = Time.monotonicNow();
          while ((!shouldStop() && dataQueue.size() == 0 &&
              (stage != BlockConstructionStage.DATA_STREAMING ||
                  now - lastPacket < halfSocketTimeout)) || doSleep) {
            long timeout = halfSocketTimeout - (now-lastPacket);
            timeout = timeout <= 0 ? 1000 : timeout;
            timeout = (stage == BlockConstructionStage.DATA_STREAMING)?
                timeout : 1000;
            try {
            //等待数据
              dataQueue.wait(timeout);
            } catch (InterruptedException  e) {
              LOG.warn("Caught exception", e);
            }
            doSleep = false;
            now = Time.monotonicNow();
          }
          if (shouldStop()) {
            continue;
          }
          // get packet to be sent.
          if (dataQueue.isEmpty()) {
              //创建一个空的心跳packet
            one = createHeartbeatPacket();
          } else {
            ...
            //
            one = dataQueue.getFirst(); // regular data packet
            ...
          }
        }

        // get new block from namenode.
        ...
        //判断是create还是append类型
        if (stage == BlockConstructionStage.PIPELINE_SETUP_CREATE) {
          LOG.debug("Allocating new block: {}", this);
          //nextBlockOutputStream：向namenode发送rpc请求获取datanode信息
          setPipeline(nextBlockOutputStream());
          //根据获取的datanode信息初始化stream
          initDataStreaming();
        } else if (stage == BlockConstructionStage.PIPELINE_SETUP_APPEND) {
          LOG.debug("Append to block {}", block);
          setupPipelineForAppendOrRecovery();
          if (streamerClosed) {
            continue;
          }
          initDataStreaming();
        }
        ...

        //如果是block中最后一个packet
        if (one.isLastPacketInBlock()) {
          // wait for all data packets have been successfully acked
          synchronized (dataQueue) {
            while (!shouldStop() && ackQueue.size() != 0) {
              try {
                // wait for acks to arrive from datanodes
                //等待响应信息
                dataQueue.wait(1000);
              }
          stage = BlockConstructionStage.PIPELINE_CLOSE;
        }
        ...
        synchronized (dataQueue) {
          // move packet from dataQueue to ackQueue
          if (!one.isHeartbeatPacket()) {
            if (scope != null) {
              spanId = scope.getSpanId();
              scope.detach();
              one.setTraceScope(scope);
            }
            scope = null;
            dataQueue.removeFirst();
            ackQueue.addLast(one);
            packetSendTime.put(one.getSeqno(), Time.monotonicNow());
            dataQueue.notifyAll();
          }
        }

        LOG.debug("{} sending {}", this, one);

        // write out data to remote datanode
        try (TraceScope ignored = dfsClient.getTracer().
            newScope("DataStreamer#writeTo", spanId)) {
          one.writeTo(blockStream);
          blockStream.flush();
        } catch (IOException e) {
          // HDFS-3398 treat primary DN is down since client is unable to
          // write to primary DN. If a failed or restarting node has already
          // been recorded by the responder, the following call will have no
          // effect. Pipeline recovery can handle only one node error at a
          // time. If the primary node fails again during the recovery, it
          // will be taken out then.
          errorState.markFirstNodeIfNotMarked();
          throw e;
        }
        lastPacket = Time.monotonicNow();

        // update bytesSent
        long tmpBytesSent = one.getLastByteOffsetBlock();
        if (bytesSent < tmpBytesSent) {
          bytesSent = tmpBytesSent;
        }

        if (shouldStop()) {
          continue;
        }

        // Is this block full?
        if (one.isLastPacketInBlock()) {
          // wait for the close packet has been acked
          synchronized (dataQueue) {
            while (!shouldStop() && ackQueue.size() != 0) {
              dataQueue.wait(1000);// wait for acks to arrive from datanodes
            }
          }
          if (shouldStop()) {
            continue;
          }

          endBlock();
        }
        if (progress != null) { progress.progress(); }

        // This is used by unit test to trigger race conditions.
        if (artificialSlowdown != 0 && dfsClient.clientRunning) {
          Thread.sleep(artificialSlowdown);
        }
      } catch (Throwable e) {
        // Log warning if there was a real error.
        if (!errorState.isRestartingNode()) {
          // Since their messages are descriptive enough, do not always
          // log a verbose stack-trace WARN for quota exceptions.
          if (e instanceof QuotaExceededException) {
            LOG.debug("DataStreamer Quota Exception", e);
          } else {
            LOG.warn("DataStreamer Exception", e);
          }
        }
        lastException.set(e);
        assert !(e instanceof NullPointerException);
        errorState.setInternalError();
        if (!errorState.isNodeMarked()) {
          // Not a datanode issue
          streamerClosed = true;
        }
      } finally {
        if (scope != null) {
          scope.close();
          scope = null;
        }
      }
    }
    closeInternal();
  } 
```
## DataStreaming创建过程：
> * 在DataStreamer.run进程中调用了nextBlockOutputStream方法
> * nextBlockOutputStream方法中调用了locateFollowingBlock方法
> * locateFollowingBlock方法调用addBlock方法从namenode获取datanode信息
> * 然后根据datanode信息创建datastreaming

**DataStreamer.run:**
```java
    if (stage == BlockConstructionStage.PIPELINE_SETUP_CREATE) {
          
          setPipeline(nextBlockOutputStream());
          initDataStreaming();
        } else if (stage == BlockConstructionStage.PIPELINE_SETUP_APPEND) {
          setupPipelineForAppendOrRecovery();
          ...
          initDataStreaming();
        }
```
**nextBlockOutputStream方法**
```java
      protected LocatedBlock nextBlockOutputStream() throws IOException {
      ...
      lb = locateFollowingBlock(
          excluded.length > 0 ? excluded : null, oldBlock);
      ...
      nodes = lb.getLocations();
      ...

      // Connect to first DataNode in the list.
      success = createBlockOutputStream(nodes, nextStorageTypes, nextStorageIDs,
          0L, false);
    ...
    return lb;
  }
```
**locateFollowingBlock方法：**
```java
    private LocatedBlock locateFollowingBlock(DatanodeInfo[] excluded,
      ExtendedBlock oldBlock) throws IOException {
    return DFSOutputStream.addBlock(excluded, dfsClient, src, oldBlock,
        stat.getFileId(), favoredNodes, addBlockFlags);
  }
```
**DFSOutputStream.addBlock方法：**
```java
    //dfsClient.namenode：ClientProtocol
    return dfsClient.namenode.addBlock(src, dfsClient.clientName, prevBlock,
            excludedNodes, fileId, favoredNodes, allocFlags);
```
**createBlockOutputStream方法：connects to the first datanode in the pipeline**
```java
      boolean createBlockOutputStream(DatanodeInfo[] nodes,
      StorageType[] nodeStorageTypes, String[] nodeStorageIDs,
      long newGS, boolean recoveryFlag) {
    ...
    while (true) {
        ...
        s = createSocketForPipeline(nodes[0], nodes.length, dfsClient);
        ...
        // send the request
        new Sender(out).writeBlock(blockCopy, nodeStorageTypes[0], accessToken,
            dfsClient.clientName, nodes, nodeStorageTypes, null, bcs,
            nodes.length, block.getNumBytes(), bytesSent, newGS,
            checksum4WriteBlock, cachingStrategy.get(), isLazyPersistFile,
            (targetPinnings != null && targetPinnings[0]), targetPinnings,
            nodeStorageIDs[0], nodeStorageIDs);

        ...
      return result;
    }
  }
```
**DataXceiver.writeBlock方法：**
```java
      //
      // Connect to downstream machine, if appropriate
      //
      if (targets.length > 0) {
        InetSocketAddress mirrorTarget = null;
        // Connect to backup machine
        mirrorNode = targets[0].getXferAddr(connectToDnViaHostname);
        LOG.debug("Connecting to datanode {}", mirrorNode);
        mirrorTarget = NetUtils.createSocketAddr(mirrorNode);
        ...
      }
```
___

## DataStreamer.ResponseProcessor进程分析
___
> * 该进程是在initDataStreaming方法中启动的 
> * ResponseProcessor是DataStreamer的内部类，DataNode接收到Packet后需要向客户端回复ACK，表示自己已经收到Packet了，而接收处理ACK的线程类就是ResponseProcessor。
> * 对每一个块的传输都需要新建一个ResponseProcessor，当块传输完，客户端会通过endBlock方法间接地把当前ResponseProcessor销毁掉。下次传输新的Block的时候通过初始化传输环境方法initDataStreaming来间接地创建ResponseProcessor。
```java
       @Override
    public void run() {

      ...
      while (!responderClosed && dfsClient.clientRunning && !isLastPacketInBlock) {
        // process responses from datanodes.
        try {
          // read an ack from the pipeline
          ack.readFields(blockReplyStream);
          ...

          long seqno = ack.getSeqno();
          // processes response status from datanodes.
          //congestedNodesFromAck中记录着处于繁忙状态的datanode信息
          ArrayList<DatanodeInfo> congestedNodesFromAck = new ArrayList<>();
          //ENC是一个枚举：DISABLED、SUPPORTED、、SUPPORTED2、CONGESTED
          for (int i = ack.getNumOfReplies()-1; i >=0  && dfsClient.clientRunning; i--) {
            final Status reply = PipelineAck.getStatusFromHeader(ack
                .getHeaderFlag(i));
            //从ack中判断是否有datanode处于CONGESTED状态
            if (PipelineAck.getECNFromHeader(ack.getHeaderFlag(i)) ==
                PipelineAck.ECN.CONGESTED) {
              congestedNodesFromAck.add(targets[i]);
            }
            // Restart will not be treated differently unless it is
            // the local node or the only one in the pipeline.
            // 检测是否有datanode处于重启状态
            if (PipelineAck.isRestartOOBStatus(reply)) {
              final String message = "Datanode " + i + " is restarting: "
                  + targets[i];
              errorState.initRestartingNode(i, message,
                  shouldWaitForRestart(i));
              throw new IOException(message);
            }
            // node error
            //得到错误的响应
            if (reply != SUCCESS) {
              errorState.setBadNodeIndex(i); // mark bad datanode
              throw new IOException("Bad response " + reply +
                  " for " + block + " from datanode " + targets[i]);
            }
          }
          ...
          // a success ack for a data packet
          DFSPacket one;
          synchronized (dataQueue) {
            one = ackQueue.getFirst();
          }
          if (one.getSeqno() != seqno) {
            throw new IOException("ResponseProcessor: Expecting seqno " +
                " for block " + block +
                one.getSeqno() + " but received " + seqno);
          }
          isLastPacketInBlock = one.isLastPacketInBlock();
            ...
          synchronized (dataQueue) {
            scope = one.getTraceScope();
            if (scope != null) {
              scope.reattach();
              one.setTraceScope(null);
            }
            lastAckedSeqno = seqno;
            pipelineRecoveryCount = 0;
            ackQueue.removeFirst();
            packetSendTime.remove(seqno);
            dataQueue.notifyAll();

            one.releaseBuffer(byteArrayManager);
          }
        } 
        ...
    }
```