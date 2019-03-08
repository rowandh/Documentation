**********************************************************************
Running the Local Smart Contracts network miner for the first time  
**********************************************************************

The Local Smart Contracts network is held in a branch on the `Stratis Full Node repository on GitHub <https://github.com/stratisproject/StratisBitcoinFullNode>`_. Make a clone of the Full Node repository on your systemn and then switch to the ``LSC-tutorial`` branch.

The Node Scripts
====================

Let's look at the scripts that run the three nodes in the ``LSC_Node_Scripts`` directory. Batch files (\*.bat) are supplied for Windows PCs, and Bash scripts (\*.sh) are supplied for Mac and Linux PCs. To give you an idea of how the the three nodes are going to interact, we are going examine the scripts in greater detail. 

Open the three scripts for your system. The scripts don't do anything other than make sure the correct branch of the GitHub repository is checked out and run one of two daemons with the required configuration options.

The miner node runs the ``Stratis.LocalSmartContracts.MinerD`` daemon, and the other two nodes run the ``Stratis.LocalSmartContracts.NodeD`` daemon. The configuration options used give us information on the topology of the network we are setting up.

Daemon configuration options
-----------------------------------------

The table below shows the configuration options used for the miner and standard nodes daemons. These options represents a small subset of the configuration options available for the Stratis Full Node.

+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Config Option | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
+===============+===================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================+
| datadir       | Specifies the directory in which to hold the data for a node. The node creates the directory structure the first time it is run. Based on settings in the scripts, you can find the data on a Windows PC in the miner1, node1, and node2 directories held within APPDATA\LocalSCNodes. On a Mac these directories are found in ~/LocalSCNodes. This overrides the standard settings which assume you will only run one node for each mainchain or sidechain on a device. In this slightly unusual situation, three nodes from one network are run on one machine. |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| port          | Specifies the port to use to interact with peers on the network.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| apiport       | Specifies the port to use for interactions via the Swagger API.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| txindex       | A flag that specifies that transactions should be saved in the database so they can be queried.                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| connect       | Specifies a peer to connect to on the network. Giving the miner an invalid IP address for this option (0) ensures it makes no attempt to connect to other peers. When the connect option is used, the node does not, by default, listen for other peers trying to connect.                                                                                                                                                                                                                                                                                        |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| listen        | Specifies the node should listen for other peers trying to connect. In the case of the miner, this flag is used to override the effect of connect which is to turn off listening.                                                                                                                                                                                                                                                                                                                                                                                 |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| addnode       | Specifies a peer (node) to connect to. When a peer is connected to using addnode, the peer sends back other nodes to connect to. For example, if node1 connects to miner1, then miner1 can, provided node2 has connected, send back node2 as a peer for node1 to connect to. The end result is that node1 is connected to miner1 and node2.                                                                                                                                                                                                                       |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| bind          | The IP address to accept data from. Specifying a local address ensures that the network remains entirely local.                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

The Local Smart Contracts Miner 
=================================

Only the miner node needs to run to create the genesis and premine blocks.

The genesis block has a timestamp and the miner will not begin mining the genesis block if the timestamp is too far in the past. The first thing to do before running the miner is to update the genesis block timestamp.

Setting the timestamp for your local smart contracts network
-------------------------------------------------------------

The timestamp is set in the `LocalSmartContracts <https://github.com/stratisproject/StratisBitcoinFullNode/blob/LSC-tutorial/src/Stratis.LocalSmartContracts.Networks/LocalSmartContractsNetwork.cs>`_ class, which inherits from `PoANetwork <https://github.com/stratisproject/StratisBitcoinFullNode/blob/LSC-tutorial/src/Stratis.Bitcoin.Features.PoA/PoANetwork.cs>`_ class and defines the LSC network. The following code excerpt from ``LocalSmartContracts()`` constructor shows the line of code where the genesis block property is set:

::

    this.GenesisTime = 1551703905;

The next chapter covers the other genesis block settings and the ``CreateLSCGenesisBlock()`` function in more detail. For now, you just need to update the ``GenesisTime`` property. The `Epoch Unix Time Stamp Converter <https://www.unixtimestamp.com>`_ provides the current UNIX timestamp:

.. image:: UNIX_Timestamp.png
     :width: 900px
     :alt: UNIX Timestamp
     :align: center

In this case, you would just update the line of code like this:

::

    this.GenesisTime = 1551958646;

Updating the expected hash values for the genesis block
--------------------------------------------------------

Now you have added the timestamp, run the miner node using the provided script. On a Windows system, use the following command:

::

    .\start_miner1.bat

On a Mac or Linux system, use:

::

    ./start_miner1.sh

The miner will run for a short time and then abort. This is because of two ``Network.Assert()`` calls, which raise an exception if a boolean condition is not met. The two conditions are as follows:

1. The hash of the genesis block must match a supplied 256 integer representing the expected hash.
2. The hash of the genesis block Merkle Root must match a supplied 256 integer representing the expected hash.

Blockchain architecture means that blocks hold a hash of the previous block, so the hash of the genesis block will be held by the premine block. Because a change of even one byte will produce a different hash, these functions check if anything unexpected has changed in the genesis block, and in this case, you have updated the timestamp.

Because you know the reason for the change, you can go ahead and update the 256 integer values. Just before the "Invalid output" line, you will notice two lines of console output similar to the following:

 
The 256 integer values will not be the same as shown above because your new timestamp will be different. Update the condition for the two assert functions:

::



Now, if any changes happen inadvertently to *your* genesis block setup, the node will not run. The updated values you see in the console output are provided by the following lines of code:

::

    Console.WriteLine("Genesis Block Hash: '{0}'", genesisBlock.GetHash().ToString());
    Console.WriteLine("Merkle Root Hash: '{0}'", genesisBlock.Header.HashMerkleRoot.ToString());

If you want, you can now comment them out. When you run the miner, the node now displays output similar to the following:

::

    ======Node stats====== 03/01/2019 14:50:11
    Headers.Height:      0        Headers.Hash:     2fa8eaac7cd4e308b447470080352b0c3a4411d10c8c11e839d5e44dffd684c7
    Consensus.Height:    0        Consensus.Hash:   2fa8eaac7cd4e308b447470080352b0c3a4411d10c8c11e839d5e44dffd684c7
    BlockStore.Height:   0        BlockStore.Hash:  2fa8eaac7cd4e308b447470080352b0c3a4411d10c8c11e839d5e44dffd684c7
    Wallet[SC].Height:   No Wallet
    
    ======Voting Manager======
    0 polls are pending, 0 polls are finished.
    0 votes are scheduled to be added to the next block this node mines.
    
    ======Connection====== agent StratisNode:0.13.0 (70012) [in:0 out:0] [recv: 0 MB sent: 0 MB]
    
    
    ======Consensus Manager======
    IBD Stage
    Chained header tree size: 0.00 MB
    Unconsumed blocks: 0 -- (0 / 200 MB). Cache is filled by: 0%
    Downloading blocks: 0 queued out of 0 pending
    
    ======Block Puller======
    Blocks being downloaded: 0
    Queued downloads: 0
    Average block size: 0 KB
    Total download speed: 0 KB/sec
    Average time to download a block: NaN ms
    Amount of blocks node can download in 1 second: NaN
    
    ======BlockStore======
    Batch Size: 0 MB / 5 MB (0 batched blocks)
    Queue Size: 0 MB (0 queued blocks)
    
    =======Mempool=======
    MempoolSize: 0    DynamicSize: 0 kb   OrphanSize: 0   
    
    ======PoA Miner======
    Mining information for the last 20 blocks.
    MISS means that miner didn't produce a block at the timestamp he was supposed to.
    ...

Now the mining node is running. However, the miner will not mine because it requires a file containing its private federation key.

Adding the federation private key
-----------------------------------

The miner's federation public key is specified in the constructor for the  `LocalSmartContracts <https://github.com/stratisproject/StratisBitcoinFullNode/blob/LSC-tutorial/src/Stratis.LocalSmartContracts.Networks/LocalSmartContractsNetwork.cs>`_ class:

::

    var federationPublicKeys = new List<PubKey>
    {
        new PubKey("02f5b2a2fc2aa9f2ab85e9727720f9b280ed937f897e444810abaada26738b13c4"),
    };

However, as we have seen, the miner is currently not mining any blocks. This is because a corresponding file containing the private key which matches the public key has not been provided. The private key is required to sign the blocks produced for the network. The file is named ``federationKey.dat``, and you can find it in the ``Federation_Key`` directory of the ``LSC-tutorial`` branch. The file is not readable as the private key is necessarily encrypted.

The federationKey.dat file will only work for the public key supplied in the `LocalSmartContracts <https://github.com/stratisproject/StratisBitcoinFullNode/blob/LSC-tutorial/src/Stratis.LocalSmartContracts.Networks/LocalSmartContractsNetwork.cs>`_ class. If you wanted to change the public key or have more miners (PoA federation members), then you can use the key generation facility. This topic is discussed in the tutorial on customizing this Local Smart Contract network.

.. note:: To shutdown a node down press ``Ctrl + C``. In order to return to the command prompt, you may have to press ``Ctrl + C`` a second time.

Now stop the miner. Copy the ``federation.dat`` file into miner1's data directory. This is specified by the ``-datadir`` command line option, and will have been created when you ran the miner node for the first time. The path on a Windows system will be something like ``C:\User\User_Name\LocalSCNodes/miner1/LocalSmartContracts/LSC``, and on a Mac or Linux system, it will be ``~/LocalSCNodes/miner1/LocalSmartContracts/LSC``. The following image shows miner1's directory structure and the ``federation.dat`` file in place. It includes the directories for node1 and node2, which you will not see until you have run them. The miner wallet file will also not be visible as no wallet has been created yet.

.. image:: Directory_Structure.png
     :width: 900px
     :alt: UNIX Timestamp
     :align: center

Once, you have copied the file over, the miner node will commence with creating the genesis block the next time it is run.
















