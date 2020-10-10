**loadFromDisk函数分析**
```java
      /**
   * Instantiates an FSNamesystem loaded from the image and edits
   * directories specified in the passed Configuration.
   * 实例化FSNamesystem,从image和edits中加载Configuration中指定的directories
   *
   * @param conf the Configuration which specifies the storage directories
   *             from which to load
   * @return an FSNamesystem which contains the loaded namespace
   * @throws IOException if loading fails
   */
  static FSNamesystem loadFromDisk(Configuration conf) throws IOException {
    // 检查配置
    checkConfiguration(conf);
    FSImage fsImage = new FSImage(conf,
        FSNamesystem.getNamespaceDirs(conf),
        FSNamesystem.getNamespaceEditsDirs(conf));
    FSNamesystem namesystem = new FSNamesystem(conf, fsImage, false);
    StartupOption startOpt = NameNode.getStartupOption(conf);
    if (startOpt == StartupOption.RECOVER) {
      namesystem.setSafeMode(SafeModeAction.SAFEMODE_ENTER);
    }

    long loadStart = monotonicNow();
    try {
      namesystem.loadFSImage(startOpt);
    } catch (IOException ioe) {
      LOG.warn("Encountered exception loading fsimage", ioe);
      fsImage.close();
      throw ioe;
    }
    long timeTakenToLoadFSImage = monotonicNow() - loadStart;
    LOG.info("Finished loading FSImage in " + timeTakenToLoadFSImage + " msecs");
    NameNodeMetrics nnMetrics = NameNode.getNameNodeMetrics();
    if (nnMetrics != null) {
      nnMetrics.setFsImageLoadTime((int) timeTakenToLoadFSImage);
    }
    namesystem.getFSDirectory().createReservedStatuses(namesystem.getCTime());
    return namesystem;
  }
```

**checkConfiguration(conf);函数分析**
```java
      /**
   * Check the supplied configuration for correctness.
   * 检查提供的配置是否正确。
   * @param conf 待验证的配置
   * @throws IOException 如果无法查询配置则抛出IO异常.
   * @throws IllegalArgumentException 如果产生不正确则抛出参数异常
   */
  private static void checkConfiguration(Configuration conf)
      throws IOException {
    //dfs.namenode.name.dir
    final Collection<URI> namespaceDirs =
        FSNamesystem.getNamespaceDirs(conf);
    //dfs.namenode.shared.edits.dir
    final Collection<URI> editsDirs =
        FSNamesystem.getNamespaceEditsDirs(conf);
    //dfs.namenode.edits.dir.required
    //dfs.namenode.shared.edits.dir
    final Collection<URI> requiredEditsDirs =
        FSNamesystem.getRequiredNamespaceEditsDirs(conf);
    //dfs.namenode.shared.edits.dir
    final Collection<URI> sharedEditsDirs =
        FSNamesystem.getSharedEditsDirs(conf);

    for (URI u : requiredEditsDirs) {
      if (u.toString().compareTo(
              DFSConfigKeys.DFS_NAMENODE_EDITS_DIR_DEFAULT) == 0) {
        continue;
      }

      // Each required directory must also be in editsDirs or in
      // sharedEditsDirs.
      if (!editsDirs.contains(u) &&
          !sharedEditsDirs.contains(u)) {
        throw new IllegalArgumentException("Required edits directory " + u
            + " not found: "
            + DFSConfigKeys.DFS_NAMENODE_EDITS_DIR_KEY + "=" + editsDirs + "; "
            + DFSConfigKeys.DFS_NAMENODE_EDITS_DIR_REQUIRED_KEY
            + "=" + requiredEditsDirs + "; "
            + DFSConfigKeys.DFS_NAMENODE_SHARED_EDITS_DIR_KEY
            + "=" + sharedEditsDirs);
      }
    }

    if (namespaceDirs.size() == 1) {
      LOG.warn("Only one image storage directory ("
          + DFS_NAMENODE_NAME_DIR_KEY + ") configured. Beware of data loss"
          + " due to lack of redundant storage directories!");
    }
    if (editsDirs.size() == 1) {
      LOG.warn("Only one namespace edits storage directory ("
          + DFS_NAMENODE_EDITS_DIR_KEY + ") configured. Beware of data loss"
          + " due to lack of redundant storage directories!");
    }
  }
```

**new FSImage函数分析**
```java
      /**
   * Construct the FSImage. Set the default checkpoint directories.
   * 构造FSImage。 设置默认检查点目录
   * 
   * Setup storage and initialize the edit log.
   * 设置存储并初始化edit log.
   *
   * @param conf Configuration
   * @param imageDirs image中存储的目录
   * @param editsDirs editlog中存储的目录.
   * @throws IOException 如果目录无效的话抛出IO异常.
   */
  protected FSImage(Configuration conf,
                    Collection<URI> imageDirs,
                    List<URI> editsDirs)
      throws IOException {
    this.conf = conf;

    storage = new NNStorage(conf, imageDirs, editsDirs);
    if(conf.getBoolean(DFSConfigKeys.DFS_NAMENODE_NAME_DIR_RESTORE_KEY,
                       DFSConfigKeys.DFS_NAMENODE_NAME_DIR_RESTORE_DEFAULT)) {
      storage.setRestoreFailedStorage(true);
    }

    this.editLog = FSEditLog.newInstance(conf, storage, editsDirs);
    archivalManager = new NNStorageRetentionManager(conf, storage, editLog);
  }
```

**namesystem.loadFSImage(startOpt)方法分析**
```java
    private void loadFSImage(StartupOption startOpt) throws IOException {
    final FSImage fsImage = getFSImage();

    // format before starting up if requested
    if (startOpt == StartupOption.FORMAT) {
      
      fsImage.format(this, fsImage.getStorage().determineClusterId());// reuse current id

      startOpt = StartupOption.REGULAR;
    }
    boolean success = false;
    writeLock();
    try {
      // 如果处于 standby state.则不调用saveNamespace方法
      MetaRecoveryContext recovery = startOpt.createRecoveryContext();
      //判断是不是旧的image
      final boolean staleImage
          = fsImage.recoverTransitionRead(startOpt, this, recovery);
      if (RollingUpgradeStartupOption.ROLLBACK.matches(startOpt)) {
        rollingUpgradeInfo = null;
      }
      final boolean needToSave = staleImage && !haEnabled && !isRollingUpgrade(); 
      LOG.info("Need to save fs image? " + needToSave
          + " (staleImage=" + staleImage + ", haEnabled=" + haEnabled
          + ", isRollingUpgrade=" + isRollingUpgrade() + ")");
      if (needToSave) {
        fsImage.saveNamespace(this);
      } else {
        // No need to save, so mark the phase done.
        StartupProgress prog = NameNode.getStartupProgress();
        prog.beginPhase(Phase.SAVING_CHECKPOINT);
        prog.endPhase(Phase.SAVING_CHECKPOINT);
      }
      // This will start a new log segment and write to the seen_txid file, so
      // we shouldn't do it when coming up in standby state
      if (!haEnabled || (haEnabled && startOpt == StartupOption.UPGRADE)
          || (haEnabled && startOpt == StartupOption.UPGRADEONLY)) {
        fsImage.openEditLogForWrite(getEffectiveLayoutVersion());
      }
      success = true;
    } finally {
      if (!success) {
        fsImage.close();
      }
      writeUnlock("loadFSImage", true);
    }
    imageLoadComplete();
  }
```
